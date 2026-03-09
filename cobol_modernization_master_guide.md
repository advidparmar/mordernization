# COBOL Modernization Master Guide
## Complete Strategy, Prompt Library & Specification Framework

**Version:** 1.0  
**Purpose:** End-to-end guide for converting a COBOL codebase into modernization-ready specifications using LLM-based analysis.  
**Compatibility:** Works with any LLM or coding agent — Claude, GPT-4, Gemini, Copilot, Cursor, Windsurf, Aider, or any API-based pipeline.

---

## Table of Contents

- **Part I: Strategy Selection**
  - Chapter 1: Codebase Assessment Dimensions
  - Chapter 2: The Six Strategies
  - Chapter 3: Strategy Scoring & Selection
  - Chapter 4: Hybrid Combinations
  - Chapter 5: Quick-Start Decision Trees

- **Part II: Artifact Framework**
  - Chapter 6: Three Output Approaches (Traditional / SDD / Dual-Format)
  - Chapter 7: Complete Artifact Map
  - Chapter 8: Cross-Referencing & Traceability System

- **Part III: Traditional Prompt Library (Human-Readable Prose)**
  - Chapter 9: Pass 1 — Discovery & Inventory Prompts
  - Chapter 10: Pass 2 — Business Specification Prompts
  - Chapter 11: Pass 3 — Functional Specification Prompts
  - Chapter 12: Pass 4 — Technical Specification Prompts
  - Chapter 13: Pass 5 — Governance & Cross-Cutting Prompts

- **Part IV: Spec-Driven Development Library (Machine-Readable Contracts)**
  - Chapter 14: Domain Model Specification (JSON Schema)
  - Chapter 15: API Contract Specification (OpenAPI 3.1)
  - Chapter 16: Business Rules Specification (YAML DSL)
  - Chapter 17: Behavior Specification (Gherkin/Cucumber)
  - Chapter 18: Process & Workflow Specification (State Machine YAML)
  - Chapter 19: Event Specification (AsyncAPI 3.0)
  - Chapter 20: Data Migration Specification (YAML Mapping)

- **Part V: Unified Dual-Format Library (Recommended)**
  - Chapter 21: Pass 1 — Dual-Format Discovery Prompts
  - Chapter 22: Pass 2 — Dual-Format Business Specification Prompts
  - Chapter 23: Pass 3 — Dual-Format Functional & API Specification Prompts
  - Chapter 24: Pass 4 — Dual-Format Technical & Behavior Specification Prompts
  - Chapter 25: Pass 5 — Dual-Format Governance Prompts

- **Part VI: Execution**
  - Chapter 26: Automation Pipeline
  - Chapter 27: Quality Gates & Checkpoints
  - Chapter 28: Effort Estimation
  - Chapter 29: Risk Checkpoints & Kill Switches
  - Chapter 30: Deliverables Checklist

---

## Placeholder Conventions Used Throughout

- `{{COBOL_SOURCE}}` — Paste or inject the COBOL program source code
- `{{COPYBOOK_SOURCE}}` — Paste or inject relevant copybook(s)
- `{{JCL_SOURCE}}` — Paste or inject JCL job streams
- `{{CICS_MAP_SOURCE}}` — Paste or inject BMS map source
- `{{PROGRAM_NAME}}` — Name of the program being analyzed
- `{{SYSTEM_NAME}}` — Name of the overall application/system
- `{{DOMAIN_CONTEXT}}` — Brief description of the business domain
- `{{PREVIOUS_PASS_OUTPUT}}` — Output from a prior pass for context
- `{{BUSINESS_SPEC}}` — Business specification from Pass 2
- `{{FUNCTIONAL_SPEC}}` — Functional specification from Pass 3
- `{{BUSINESS_RULES_SPEC}}` — Machine-readable rules YAML from Pass 2
- `{{API_SPEC}}` — OpenAPI spec from Pass 3
- `{{DOMAIN_MODEL_SPEC}}` — JSON Schema domain model from Pass 1
- `{{INVENTORY}}` — Inventory output from Pass 1

---

# PART I: STRATEGY SELECTION

---

## Chapter 1: Codebase Assessment Dimensions

Answer each question honestly. The framework only helps if the inputs are accurate.

### Dimension A: Scale

| Score | Programs | Lines of Code | Typical Profile |
|-------|----------|--------------|-----------------|
| 1 | Under 50 | Under 50K LOC | Small departmental system |
| 2 | 50-200 | 50K-250K LOC | Mid-size application |
| 3 | 200-500 | 250K-1M LOC | Large application, multiple business functions |
| 4 | 500-1,000 | 1M-3M LOC | Enterprise system, core business platform |
| 5 | 1,000+ | 3M+ LOC | Massive legacy estate, multiple systems |

### Dimension B: Complexity & Coupling

| Score | Characteristics |
|-------|----------------|
| 1 | Programs are mostly independent. Few CALL statements. Clean separation. Each program does one thing. |
| 2 | Some shared copybooks and subroutines. Programs cluster naturally into 3-5 obvious groups. |
| 3 | Moderate interdependencies. Shared copybooks are common. Some programs called by 10+ others. |
| 4 | Heavy coupling. Extensive shared WORKING-STORAGE via copybooks. Dynamic CALLs. Programs 5,000+ lines. |
| 5 | Deeply entangled monolith. Circular dependencies. Programs modify shared resources in undocumented ways. |

### Dimension C: Business Criticality & Regulatory Exposure

| Score | Characteristics |
|-------|----------------|
| 1 | Internal tool. Minor differences acceptable. No regulatory oversight. |
| 2 | Customer-facing but non-financial. Errors cause inconvenience, not financial loss. |
| 3 | Financial system but not directly regulated. Penny-level precision matters. |
| 4 | Regulated industry. External auditors review changes. Documented specs legally required. |
| 5 | Safety-critical or heavily regulated. SOX, Basel III, HIPAA. Regulatory approval required. |

### Dimension D: Knowledge Availability

| Score | Characteristics |
|-------|----------------|
| 1 | Original developers available. Comprehensive documentation. SMEs understand everything. |
| 2 | Some original developers remain. Documentation partially outdated. SMEs know major processes. |
| 3 | Original developers gone but some COBOL-literate staff remain. Documentation sparse. |
| 4 | No COBOL expertise in-house. No documentation. SMEs know screens and reports but not rules. |
| 5 | Nobody understands the system. Complete black box. Last knowledgeable person left years ago. |

### Dimension E: Timeline & Business Pressure

| Score | Characteristics |
|-------|----------------|
| 1 | No time pressure. 2-3 year horizon. Mainframe contract stable. |
| 2 | Moderate pressure. Want visible progress within 6-12 months. |
| 3 | Significant pressure. Mainframe renewal in 12-18 months. Need ROI quickly. |
| 4 | Urgent. Contract ending or last COBOL developer retiring. Off mainframe within 12 months. |
| 5 | Emergency. System failing or blocking critical business initiatives. Every month has measurable cost. |

### Dimension F: Target Architecture Clarity

| Score | Characteristics |
|-------|----------------|
| 1 | No target architecture. "We just need to get off COBOL." |
| 2 | High-level direction chosen (e.g., "Java on cloud") but no detail. |
| 3 | Target architecture defined. Stack chosen. Service boundaries roughly sketched. |
| 4 | Detailed architecture with service decomposition, API patterns, data architecture. |
| 5 | Architecture defined AND validated with proof-of-concept. Team trained. CI/CD ready. |

### Dimension G: Test Infrastructure

| Score | Characteristics |
|-------|----------------|
| 1 | No automated tests. No recorded test data. Testing is manual and ad-hoc. |
| 2 | Some test JCL with canned data. Manual test scripts for major scenarios. |
| 3 | Reasonable test data sets exist. Can capture production I/O for batch. Some automation. |
| 4 | Comprehensive test data. Can record/replay production transactions. Automated regression. |
| 5 | Full test infra with production-like data, parallel-run capability, comparison tooling. |

---

## Chapter 2: The Six Strategies

### Strategy 1: Multi-Pass Sequential

Multiple structured passes through the ENTIRE codebase, each pass focusing on a different layer (inventory → business → functional → technical → governance).

**Strengths:** Most thorough. Best cross-program coverage. Highest audit/compliance readiness. Catches business rule contradictions across programs.

**Weaknesses:** Slowest time to first Java code. Risk of analysis paralysis. Specs can become outdated before development starts. Stakeholder patience wears thin.

**Best for:** Regulated industries. Mission-critical systems. Large codebases where completeness matters more than speed.

**Effort for 500 programs:** 4-6 months of analysis before development begins.

### Strategy 2: Program-at-a-Time Single Pass

Take ONE program at a time and extract everything (inventory, business rules, functional spec, technical spec, tests, migration mapping) in a single prompt.

**Strengths:** Fast start. Each analyzed program can go to development immediately. Simple to parallelize. Works well for smaller codebases.

**Weaknesses:** Loses the cross-program view. Can't detect contradictions between programs. Dependency graphs and service decomposition require global view. Quality degrades for very large programs.

**Best for:** Small codebases (under 200 programs). Low coupling. When you want to start development quickly.

**Effort for 500 programs:** 2-4 months.

### Strategy 3: Domain-Driven Decomposition First

Identify business domains/bounded contexts FIRST by analyzing naming conventions, copybook sharing, DB2 tables, and JCL groupings. Then analyze one domain at a time as a complete unit.

**Strengths:** Aligns analysis with target architecture. Catches cross-program issues within each domain. Produces coherent service definitions naturally. Arguably the best approach for modernization.

**Weaknesses:** Fails if the system is a deeply tangled monolith with no clean domain boundaries. Can't work if domain knowledge is too scarce to identify contexts.

**Best for:** Well-structured modernization. Medium-to-large codebases with identifiable business domains. Teams with defined target architecture.

**Effort for 500 programs:** 3-5 months.

### Strategy 4: Strangler Fig — Analyze Only What You're Replacing Next

Don't analyze the entire codebase. Only analyze the programs you're replacing in the next sprint/release. Wrap the rest behind an anti-corruption layer. Modernize incrementally.

**Strengths:** Fastest time to deployed Java code. Proves approach on small module before full commitment. Shows incremental progress. How most successful large-scale modernizations actually work.

**Weaknesses:** Pick the wrong starting module and you hit a wall of dependencies. Anti-corruption layer between old and new can be complex. Total effort may exceed comprehensive analysis due to repeated boundary work.

**Best for:** Large codebases. When business can't wait for analysis to complete. When you need to prove value quickly.

**Effort for 500 programs:** 6-18 months (analysis + development interleaved).

### Strategy 5: AI-Assisted Parallel Conversion

Use the LLM to BOTH analyze COBOL and produce Java implementation simultaneously. Specs generated as documentation of what was built, not plans for what to build.

**Strengths:** Fastest time to working code. For simple COBOL (CRUD, reports), LLM can produce decent Java that needs refinement. High throughput.

**Weaknesses:** Highest risk. No spec to validate against. Complex business logic may be subtly wrong. Auditors and compliance teams often reject this. Precision/rounding differences won't be caught without specs.

**Best for:** Low-risk, high-volume "commodity" programs. The "long tail" of simple utilities after critical programs are handled rigorously.

**Effort for 500 programs:** 1-3 months (but heavy review needed).

### Strategy 6: Test-First Reverse Engineering

Before analyzing code, capture actual runtime behavior by recording production inputs and outputs. These recordings become your test suite. Write Java that passes these tests, analyzing COBOL only when needed to understand specific behaviors.

**Strengths:** Doesn't require understanding COBOL internals. Ensures behavioral equivalence by definition. Works for black-box systems nobody understands. Powerful with property-based testing.

**Weaknesses:** Production data may not cover edge cases. Encodes existing bugs as expected behavior. Doesn't produce documented specs (regulatory concern). Requires infrastructure for recording/replay.

**Best for:** Black-box systems with good production data. When behavioral equivalence matters more than understanding internals.

**Effort for 500 programs:** 3-6 months (including capture phase).

---

## Chapter 3: Strategy Scoring & Selection

### Scoring Tables

For each strategy below, find your dimension score (1-5 from Chapter 1) and look up the corresponding Fit value. Multiply Fit × Weight for each row, then sum.

#### Strategy 1: Multi-Pass Sequential

| Dimension | Weight | Fit@1 | Fit@2 | Fit@3 | Fit@4 | Fit@5 |
|-----------|--------|-------|-------|-------|-------|-------|
| A: Scale | 2 | 5 | 5 | 4 | 3 | 2 |
| B: Complexity | 3 | 3 | 4 | 5 | 5 | 4 |
| C: Criticality | 4 | 2 | 3 | 4 | 5 | 5 |
| D: Knowledge | 2 | 5 | 4 | 3 | 3 | 2 |
| E: Timeline | 3 | 5 | 4 | 3 | 1 | 1 |
| F: Architecture | 1 | 3 | 4 | 4 | 5 | 5 |
| G: Test Infra | 1 | 3 | 3 | 4 | 4 | 5 |

**Max possible: 80**

#### Strategy 2: Single-Pass Deep

| Dimension | Weight | Fit@1 | Fit@2 | Fit@3 | Fit@4 | Fit@5 |
|-----------|--------|-------|-------|-------|-------|-------|
| A: Scale | 2 | 5 | 5 | 3 | 2 | 1 |
| B: Complexity | 3 | 5 | 4 | 3 | 2 | 1 |
| C: Criticality | 3 | 5 | 4 | 3 | 2 | 1 |
| D: Knowledge | 2 | 4 | 4 | 3 | 3 | 3 |
| E: Timeline | 3 | 3 | 4 | 5 | 5 | 4 |
| F: Architecture | 2 | 2 | 3 | 4 | 5 | 5 |
| G: Test Infra | 1 | 3 | 3 | 4 | 4 | 5 |

**Max possible: 80**

#### Strategy 3: Domain-Driven Decomposition

| Dimension | Weight | Fit@1 | Fit@2 | Fit@3 | Fit@4 | Fit@5 |
|-----------|--------|-------|-------|-------|-------|-------|
| A: Scale | 2 | 4 | 5 | 5 | 4 | 3 |
| B: Complexity | 3 | 3 | 4 | 5 | 4 | 3 |
| C: Criticality | 3 | 3 | 4 | 5 | 5 | 4 |
| D: Knowledge | 2 | 5 | 4 | 4 | 3 | 2 |
| E: Timeline | 2 | 5 | 4 | 4 | 3 | 2 |
| F: Architecture | 3 | 1 | 2 | 4 | 5 | 5 |
| G: Test Infra | 1 | 3 | 3 | 4 | 4 | 5 |

**Max possible: 80**

#### Strategy 4: Strangler Fig

| Dimension | Weight | Fit@1 | Fit@2 | Fit@3 | Fit@4 | Fit@5 |
|-----------|--------|-------|-------|-------|-------|-------|
| A: Scale | 2 | 3 | 4 | 4 | 5 | 5 |
| B: Complexity | 2 | 4 | 4 | 3 | 3 | 2 |
| C: Criticality | 3 | 3 | 4 | 4 | 4 | 3 |
| D: Knowledge | 2 | 4 | 4 | 4 | 3 | 3 |
| E: Timeline | 4 | 2 | 3 | 4 | 5 | 5 |
| F: Architecture | 2 | 2 | 3 | 4 | 5 | 5 |
| G: Test Infra | 1 | 3 | 3 | 4 | 5 | 5 |

**Max possible: 80**

#### Strategy 5: AI-Assisted Parallel Conversion

| Dimension | Weight | Fit@1 | Fit@2 | Fit@3 | Fit@4 | Fit@5 |
|-----------|--------|-------|-------|-------|-------|-------|
| A: Scale | 2 | 5 | 4 | 4 | 3 | 3 |
| B: Complexity | 3 | 5 | 4 | 3 | 1 | 1 |
| C: Criticality | 4 | 5 | 4 | 2 | 1 | 1 |
| D: Knowledge | 1 | 4 | 4 | 3 | 3 | 3 |
| E: Timeline | 3 | 3 | 4 | 5 | 5 | 5 |
| F: Architecture | 2 | 2 | 3 | 4 | 5 | 5 |
| G: Test Infra | 2 | 1 | 2 | 3 | 5 | 5 |

**Max possible: 85**

#### Strategy 6: Test-First Reverse Engineering

| Dimension | Weight | Fit@1 | Fit@2 | Fit@3 | Fit@4 | Fit@5 |
|-----------|--------|-------|-------|-------|-------|-------|
| A: Scale | 1 | 4 | 4 | 4 | 3 | 3 |
| B: Complexity | 2 | 4 | 4 | 3 | 3 | 4 |
| C: Criticality | 3 | 3 | 3 | 3 | 3 | 2 |
| D: Knowledge | 3 | 2 | 3 | 4 | 5 | 5 |
| E: Timeline | 2 | 3 | 4 | 4 | 4 | 3 |
| F: Architecture | 1 | 3 | 3 | 4 | 4 | 5 |
| G: Test Infra | 4 | 1 | 2 | 3 | 5 | 5 |

**Max possible: 80**

### Interpretation Rules

**One strategy scores 20%+ higher than all others:** Use it as your primary approach.

**Two strategies score within 10% of each other:** Use a hybrid (see Chapter 4).

**Three or more within 10%:** Complex situation — hybrid is essential (see Chapter 4).

**No strategy scores above 50%:** Unusual characteristics — see fallback guidance in Chapter 4.

---

## Chapter 4: Hybrid Combinations

### Hybrid A: "Rigorous Start, Accelerating Finish"

**Combine:** Strategy 1 (Multi-Pass) + Strategy 5 (Parallel Conversion)

**When:** Criticality is high (4-5) but Timeline pressure is also high (3-4).

**How:**
- Multi-Pass for the top 20% most critical/complex programs
- Parallel Conversion for the remaining 80% commodity programs
- A program qualifies for Parallel Conversion only if its complexity is low AND it has no financial calculations AND it has fewer than 3 external dependencies

### Hybrid B: "Domain Strangler"

**Combine:** Strategy 3 (Domain-Driven) + Strategy 4 (Strangler Fig)

**When:** Scale is large (3-5) and Architecture clarity is good (3-5).

**How:**
- Domain-Driven Decomposition to identify all bounded contexts
- Strangler Fig to modernize one domain at a time
- Each domain gets full dual-format analysis just before development
- Don't analyze domains you won't touch for 6+ months

### Hybrid C: "Test-Driven Discovery"

**Combine:** Strategy 6 (Test-First) + Strategy 3 (Domain-Driven)

**When:** Knowledge is low (4-5) and Test infrastructure is good (3-5).

**How:**
- Capture production behavior first (Test-First)
- Use captured data to understand what programs actually do
- Domain-Driven analysis with captured data as validation
- For programs where captured behavior is insufficient, fall back to LLM code analysis

### Hybrid D: "Comprehensive with Safety Net"

**Combine:** Strategy 3 (Domain-Driven) + Strategy 6 (Test-First) + Strategy 1 (Multi-Pass for critical paths)

**When:** Regulated industry. Can't afford to get this wrong.

**How:**
- Phase 1 (2 weeks): Lightweight inventory scan (Strategy 1, Pass 1 only)
- Phase 2 (2 weeks): Capture production behavior for critical paths (Strategy 6)
- Phase 3 (ongoing): Domain-by-domain deep analysis and development (Strategy 3)
- Phase 4 (per domain): Multi-Pass deep dive only for programs with financial calculations, regulatory logic, or complex state machines
- Safety net: Run captured tests against every modernized component

### Hybrid E: "Fast Start, Rigorous Core"

**Combine:** Strategy 4 (Strangler Fig) + Strategy 2 (Single-Pass) + Strategy 1 (Multi-Pass for core)

**When:** Timeline is extreme (4-5) but some parts are high criticality.

**How:**
- Week 1-2: Quick inventory scan, identify most isolated module
- Week 3-4: Single-Pass Deep Dive on first module, build Java, deploy
- Month 2-3: Repeat for 2-3 more isolated modules (building confidence)
- Month 4+: Multi-Pass thorough analysis for core complex modules
- Early wins buy time and credibility for the harder analysis

### Universal Hybrid (When Scores Are Close)

```
Week 1-2:   SCAN (lightweight, all programs)
            → Output: Inventory, dependency graph, domain clusters
            
Week 3:     PRIORITIZE
            → TIER 1 (Critical): Financial calcs, regulatory, complex
            → TIER 2 (Important): Business logic, moderate complexity
            → TIER 3 (Commodity): CRUD, simple reports, utilities
            → Select first domain to modernize

Week 4-6:   DEEP DIVE (first domain, Tier 1 programs)
            → Full dual-format analysis

Week 7-8:   BUILD & VALIDATE (first domain)
            → Java development from specs
            → Run Gherkin tests, parallel run against COBOL

Week 9+:    REPEAT for next domain
            → Tier 1: Full dual-format analysis
            → Tier 2: Single-Pass Deep Dive
            → Tier 3: AI-Assisted Parallel Conversion
```

### Fallback Guidance

**High criticality + Low knowledge + Urgent timeline:** Invest heavily in Test-First (capture production behavior) BEFORE any code analysis. Then Domain-Driven with aggressive SME engagement. Hire COBOL consultants if needed.

**Massive scale + Deep complexity + No architecture vision:** Stop. Invest 4-6 weeks in defining target architecture first. Run a proof-of-concept. Only THEN analyze the code.

**Low criticality + Low knowledge + No test infrastructure:** Use Parallel Conversion with manual code review. Accept higher risk — for non-critical systems, fixing issues in production costs less than extensive pre-analysis.

---

## Chapter 5: Quick-Start Decision Trees

### Tree 1: "I have 500 COBOL programs. Where do I start?"

```
Is your timeline urgent (< 6 months to show results)?
├── YES → Do you have a target architecture defined?
│   ├── YES → Hybrid B (Domain Strangler)
│   └── NO  → Spend 2 weeks on architecture FIRST
│             Then Hybrid E (Fast Start, Rigorous Core)
└── NO  → Is the system regulated / high criticality?
    ├── YES → Hybrid D (Comprehensive with Safety Net)
    └── NO  → Strategy 3 (Domain-Driven Decomposition)
```

### Tree 2: "I have 50 COBOL programs. Fastest path?"

```
Are there financial calculations with precision requirements?
├── YES → Strategy 2 (Single-Pass Deep) with dual-format
│         Extra attention to precision mapping
└── NO  → Is the code well-structured (low complexity)?
    ├── YES → Strategy 5 (AI-Assisted Parallel Conversion)
    └── NO  → Strategy 2 (Single-Pass Deep)
```

### Tree 3: "Nobody understands our COBOL system."

```
Can you capture production inputs/outputs?
├── YES → Hybrid C (Test-Driven Discovery)
└── NO  → Can you hire COBOL consultants?
    ├── YES → Strategy 1 (Multi-Pass Sequential)
    └── NO  → Strategy 2 (Single-Pass Deep)
              Flag EVERYTHING as LOW confidence
```

### Tree 4: "We're in a regulated industry."

```
Do you need documented specifications for audit?
├── YES → Use Dual-Format prompts (Part V of this guide)
│         Strategy 1 or 3 depending on scale
└── NO  → Strategy 3 (Domain-Driven Decomposition)
          Still maintain traceability
```

---

# PART II: ARTIFACT FRAMEWORK

---

## Chapter 6: Three Output Approaches

This guide provides prompts for three approaches. Choose based on your needs:

### Approach 1: Traditional (Human-Readable Prose Only)

Produces Word-doc-style specifications: narrative descriptions, tables, and structured text.

**Artifacts produced:** 18 documents across 5 passes.

**Use when:** Your team consumes documents. No code generation tooling. Stakeholders need to read and approve prose. Regulatory audit requires narrative documentation.

**Prompts:** Part III (Chapters 9-13)

### Approach 2: Spec-Driven Development (Machine-Readable Only)

Produces machine-parseable specs: JSON Schema, OpenAPI, YAML DSL, Gherkin, AsyncAPI.

**Artifacts produced:** 7 spec types.

**Use when:** You have code generation tooling. CI/CD will enforce contract conformance. Development team prefers executable specs over documents. You want specs that never go stale.

**Prompts:** Part IV (Chapters 14-20)

### Approach 3: Unified Dual-Format (Both — Recommended)

Produces BOTH human-readable AND machine-readable from a single LLM pass. Output is split at a marker into two streams.

**Artifacts produced:** 12+ paired artifacts (human + machine for each).

**Use when:** You need stakeholder approval (human docs) AND development automation (machine specs). Regulated industry with modern development practices. Best of both worlds.

**Prompts:** Part V (Chapters 21-25)

---

## Chapter 7: Complete Artifact Map

### Traditional Artifacts (18 total)

| Pass | Artifact | Audience |
|------|----------|----------|
| 1 | Application Inventory Catalog | Project managers, architects |
| 1 | Dependency & Call Graph Map | Architects, developers |
| 1 | Data Inventory & Dictionary | Data architects, analysts |
| 1 | JCL Job Stream Analysis | Operations, batch developers |
| 2 | Business Process Documents | Business SMEs, product owners |
| 2 | Business Rules Catalog | Business analysts, compliance |
| 3 | Functional Specification Documents | Dev leads, Java developers |
| 3 | Screen/UI Mapping Document | UX designers, frontend devs |
| 3 | Report & Output Specifications | Business users, BI team |
| 3 | Interface & Integration Specifications | Integration architects |
| 4 | Low-Level Technical Specifications | Java developers |
| 4 | Data Model Design Document | Data architects, DBAs |
| 4 | Service Decomposition Map | Solution architects |
| 4 | Error Handling & Logging Strategy | Java developers, SREs |
| 4 | Batch Processing Design Document | Java developers, operations |
| 5 | Traceability Matrix | QA leads, auditors |
| 5 | Gap & Risk Register | Risk committee, project leads |
| 5 | Test Scenario Catalog | QA architects, test leads |

### SDD Artifacts (7 total)

| Artifact | Format | Replaces (Traditional) | Consumed By |
|----------|--------|----------------------|-------------|
| Domain Model Spec | JSON Schema + YAML | Data Dictionary, Data Model | Code generators, ORM, DB migration |
| API Contract Spec | OpenAPI 3.1 | Functional Spec, Interface Spec | Server stubs, client SDKs, API docs |
| Business Rules Spec | YAML DSL | Business Rules Catalog | Rule engines, validation generators |
| Behavior Spec | Gherkin | Test Scenario Catalog | Cucumber test runners |
| Process Spec | State Machine YAML | Business Process Document | Workflow engines |
| Event Spec | AsyncAPI 3.0 | Interface Spec (async) | Event framework config |
| Migration Spec | YAML mapping | Data Migration sections | ETL pipeline generators |

### Dual-Format Artifacts (Paired)

| Pass | Human-Readable | Machine-Readable |
|------|---------------|-----------------|
| 1 | Program Inventory Summary | inventory.yaml |
| 1 | Data Dictionary | domain-model.schema.yaml |
| 2 | Business Process Document | workflow.yaml |
| 2 | Business Rules Narrative | rules.yaml |
| 3 | Functional Specification | openapi.yaml |
| 3 | Interface Specification | interface.yaml + asyncapi.yaml |
| 4 | Technical Specification | feature files (Gherkin) |
| 4 | Data Migration Notes | migration.mapping.yaml |
| 5 | Governance Report | traceability-matrix.yaml |
| 5 | Risk Narrative | risk-register.yaml |

---

## Chapter 8: Cross-Referencing & Traceability System

### ID Conventions

Every artifact uses consistent IDs that link human and machine formats:

- **BR-{category}-{number}** — Business rules (e.g., BR-V-001, BR-C-015)
  - Categories: V=Validation, C=Calculation, E=Eligibility, T=Threshold, R=Routing, S=State transition
- **FS-{module}-{number}** — Functional specs (e.g., FS-LENDING-001)
- **TS-{program}** — Technical specs (e.g., TS-INTCALC)
- **TC-{category}-{number}** — Test cases (e.g., TC-HP-001)
  - Categories: HP=Happy path, BC=Boundary, ER=Error, PR=Precision, BA=Batch, RG=Regression
- **INT-{number}** — Interfaces (e.g., INT-001)
- **MIG-{entity}** — Migration mappings (e.g., MIG-LOAN-ACCOUNT)
- **WF-{process}** — Workflows (e.g., WF-LOAN-LIFECYCLE)
- **EVT-{domain}-{event}** — Events (e.g., EVT-LENDING-INTEREST-CALCULATED)
- **RISK-{number}** — Risks (e.g., RISK-001)
- **ASMP-{number}** — Assumptions (e.g., ASMP-001)
- **TRC-{number}** — Traceability entries (e.g., TRC-001)

### How Artifacts Link Together

```
HUMAN DOCUMENT                 LINKS VIA                   MACHINE SPEC
─────────────────────────────────────────────────────────────────────────
Inventory Summary         ←── x-human-readable-doc ──→  inventory.yaml
Business Process Doc      ←── x-human-readable-doc ──→  workflow.yaml
Business Rules Table      ←── id: BR-C-001 ──────────→  rules.yaml
Functional Spec §4.2      ←── x-human-readable-doc ──→  openapi.yaml
Technical Spec §3         ←── x-human-readable-doc ──→  feature files
Governance Report         ←── trace_id: TRC-001 ────→  traceability.yaml
```

---

# PART III: TRADITIONAL PROMPT LIBRARY (Human-Readable Prose)

---

## Chapter 9: Pass 1 — Discovery & Inventory Prompts

### Prompt 1.1 — Application Inventory Catalog

```
ROLE: You are a COBOL mainframe analyst performing an inventory assessment of a legacy system.

CONTEXT:
- System: {{SYSTEM_NAME}}
- Domain: {{DOMAIN_CONTEXT}}

TASK: Analyze the following COBOL program and produce an inventory entry.

SOURCE CODE:
{{COBOL_SOURCE}}

COPYBOOKS USED:
{{COPYBOOK_SOURCE}}

OUTPUT FORMAT (respond ONLY in this exact structure):

## INVENTORY ENTRY

**Program ID:** [Program name from PROGRAM-ID]
**Source File:** [Filename if known]
**Type:** [BATCH | ONLINE/CICS | SUBROUTINE | UTILITY | COPYBOOK | OTHER]
**Lines of Code:** [Approximate count]
**One-Line Summary:** [Single sentence describing what this program does]
**Division Summary:**
- IDENTIFICATION: [Program-ID, author, date if present]
- ENVIRONMENT: [File assignments, special names]
- DATA: [Number of FDs, WS items, LINKAGE items]
- PROCEDURE: [Number of sections/paragraphs, high-level flow]

**External Dependencies:**
- Programs Called: [List all CALL statements with program names]
- Called By: [If known from context, otherwise "Unknown — requires cross-reference"]
- Copybooks Used: [List all COPY statements]
- Files Accessed: [List all FD/SD entries with SELECT assignments]
- DB2 Tables: [List all tables from EXEC SQL statements]
- CICS Resources: [Maps, queues, TDQs, TSQs if applicable]
- MQ Queues: [If applicable]

**Technology Footprint:**
- Database Access: [DB2 | VSAM | Sequential | IMS DB | None]
- Transaction Monitor: [CICS | IMS TM | Batch | None]
- Middleware: [MQ | CICS TD/TS | None]
- Report Writer: [Yes/No, tool if known]

**Preliminary Complexity Assessment:**
- Complexity: [LOW | MEDIUM | HIGH | VERY HIGH]
- Rationale: [Brief explanation]

**Notes/Flags:**
[Any unusual patterns, dead code indicators, commented-out sections, or items needing human review]
```

### Prompt 1.2 — Dependency & Call Graph Extraction

```
ROLE: You are a COBOL mainframe analyst building a dependency graph for a legacy system.

CONTEXT:
- System: {{SYSTEM_NAME}}
- Domain: {{DOMAIN_CONTEXT}}

TASK: Analyze the following COBOL program and extract ALL dependency relationships. Be exhaustive — missing a dependency here means missing it in the entire modernization.

SOURCE CODE:
{{COBOL_SOURCE}}

COPYBOOKS USED:
{{COPYBOOK_SOURCE}}

JCL (if available):
{{JCL_SOURCE}}

OUTPUT FORMAT (respond ONLY in this exact structure):

## DEPENDENCY EXTRACT — {{PROGRAM_NAME}}

### Program-to-Program Calls
| Called Program | Call Type | Parameters Passed | Paragraph/Section Where Called |
|---|---|---|---|
| [program name] | [STATIC CALL / DYNAMIC CALL / CICS LINK / CICS XCTL] | [List USING parameters] | [Where in the code] |

### Copybook References
| Copybook Name | Used In Division | Purpose |
|---|---|---|
| [copybook name] | [DATA / PROCEDURE / both] | [Brief description] |

### File I/O Dependencies
| File Name (FD) | DD Name (SELECT) | Access Mode | Record Format | Operations |
|---|---|---|---|---|
| [FD name] | [DD/assignment name] | [SEQUENTIAL / RANDOM / DYNAMIC] | [FB/VB/etc.] | [READ / WRITE / REWRITE / DELETE] |

### Database Dependencies
| Table/View Name | Operation | SQL Type | Cursor Name | Key Columns Used |
|---|---|---|---|---|
| [table name] | [SELECT / INSERT / UPDATE / DELETE] | [Static / Dynamic] | [cursor name or N/A] | [WHERE clause columns] |

### CICS Resource Dependencies
| Resource Type | Resource Name | Operation |
|---|---|---|
| [MAP / TDQ / TSQ / FILE / PROGRAM] | [resource name] | [SEND / RECEIVE / WRITEQ / READQ / etc.] |

### MQ Dependencies (if applicable)
| Queue Name | Operation | Message Format |
|---|---|---|
| [queue name] | [PUT / GET] | [copybook or format used] |

### Data Flow Summary
- **Inputs:** [List all data sources]
- **Outputs:** [List all data targets]
- **Pass-Through Data:** [Data received via LINKAGE and passed on]
```

### Prompt 1.3 — Data Inventory & Dictionary

```
ROLE: You are a COBOL data analyst cataloging all data structures in a legacy system.

CONTEXT:
- System: {{SYSTEM_NAME}}
- Domain: {{DOMAIN_CONTEXT}}

TASK: Extract a complete data dictionary from the following COBOL source and copybooks.

SOURCE CODE:
{{COBOL_SOURCE}}

COPYBOOKS:
{{COPYBOOK_SOURCE}}

OUTPUT FORMAT:

## DATA DICTIONARY — {{PROGRAM_NAME}}

### File Record Layouts
For each FD in the program:

**File: [FD Name] — [DD Name/Assignment]**
| Level | Field Name | PIC Clause | Type | Size (bytes) | Occurs | Redefines | Business Purpose |
|---|---|---|---|---|---|---|---|
| [01/05/10/etc.] | [field name] | [PIC clause] | [Alpha/Numeric/Packed/Binary] | [storage size] | [OCCURS count or N/A] | [field name or N/A] | [Inferred business meaning] |

### Working Storage Structures
For each 01-level group in WORKING-STORAGE:

**Structure: [01-level name] — [Purpose]**
| Level | Field Name | PIC Clause | Type | Size | Initial Value | Business Purpose |
|---|---|---|---|---|---|---|
| [level] | [field name] | [PIC clause] | [type] | [size] | [VALUE clause or N/A] | [Inferred meaning] |

### LINKAGE SECTION Parameters
| Level | Field Name | PIC Clause | Type | Size | Direction | Business Purpose |
|---|---|---|---|---|---|---|
| [level] | [field name] | [PIC clause] | [type] | [size] | [IN / OUT / IN-OUT] | [meaning] |

### Condition Names (88-levels)
| Parent Field | Condition Name | Value(s) | Business Meaning |
|---|---|---|---|
| [parent field] | [88-level name] | [VALUE clause] | [What this condition represents] |

### Key Constants and Literals
| Location | Value | Context of Use | Likely Business Meaning |
|---|---|---|---|
| [paragraph/section] | [literal value] | [How it's used] | [What it probably means] |

### REDEFINES Analysis
| Original Field | Redefining Field | Purpose |
|---|---|---|
| [original] | [redefining] | [Why the redefinition exists] |

### Data Type Translation Reference
| COBOL PIC | Storage | Suggested Java Type | Notes |
|---|---|---|---|
| [PIC clause] | [size in bytes] | [Java equivalent] | [Precision concerns, sign handling] |
```

### Prompt 1.4 — JCL Job Stream Analysis

```
ROLE: You are a mainframe operations analyst documenting batch job streams.

CONTEXT:
- System: {{SYSTEM_NAME}}
- Domain: {{DOMAIN_CONTEXT}}

TASK: Analyze the following JCL and document the complete job stream.

JCL SOURCE:
{{JCL_SOURCE}}

OUTPUT FORMAT:

## JOB STREAM ANALYSIS — [Job Name]

### Job Overview
- **Job Name:** [JOB card name]
- **Job Class:** [CLASS parameter]
- **Description:** [What this job does end-to-end]
- **Typical Schedule:** [If determinable]

### Step-by-Step Breakdown
For each EXEC step:

**Step [number]: [Step Name]**
- **Program:** [PGM= value]
- **Condition:** [COND= parameter — what conditions skip this step]
- **Purpose:** [What this step accomplishes]
- **Input Files:**
  | DD Name | DSN | DISP | Description |
  |---|---|---|---|
  | [DD name] | [dataset name] | [DISP values] | [purpose] |
- **Output Files:**
  | DD Name | DSN | DISP | Space | Description |
  |---|---|---|---|---|
  | [DD name] | [dataset name] | [DISP values] | [SPACE] | [purpose] |
- **SYSIN Parameters:** [Any inline parameters or control cards]

### Data Flow Diagram (Text)
[Describe flow of data between steps]

### Restart/Recovery Considerations
- **Restart Points:** [Which steps can be restarted independently]
- **Manual Intervention Points:** [Steps requiring operator action]

### Condition Code Logic
| Step | Expected RC | Action on RC > 0 | Impact on Subsequent Steps |
|---|---|---|---|
| [step name] | [expected return code] | [what happens] | [which steps are skipped/affected] |
```

---

## Chapter 10: Pass 2 — Business Specification Prompts

### Prompt 2.1 — Business Process Document

```
ROLE: You are a senior business analyst reverse-engineering business processes from legacy COBOL code. You write for a business audience — no technical jargon, no COBOL terminology.

CONTEXT:
- System: {{SYSTEM_NAME}}
- Domain: {{DOMAIN_CONTEXT}}
- Related programs in this process: [list if known from Pass 1]

TASK: Analyze the following COBOL program(s) and produce a business process document. Focus ONLY on WHAT the business process does and WHY — not HOW it's implemented.

SOURCE CODE:
{{COBOL_SOURCE}}

COPYBOOKS:
{{COPYBOOK_SOURCE}}

INVENTORY CONTEXT (from Pass 1):
{{PREVIOUS_PASS_OUTPUT}}

OUTPUT FORMAT:

## BUSINESS PROCESS DOCUMENT

### Process Name
[Business-friendly name — e.g., "Monthly Interest Calculation" not "INTCALC Batch Program"]

### Business Objective
[2-3 sentences: What business goal does this process achieve?]

### Business Context
[Where does this process fit in the overall business workflow?]

### Actors & Stakeholders
| Actor | Role in Process |
|---|---|
| [e.g., "Loan Officer"] | [What they do or receive] |

### Process Flow (Business Language)
1. [First business step]
2. [Next step]
3. [Continue for all steps...]

### Business Rules
| Rule ID | Rule Description | Category | Source Location |
|---|---|---|---|
| BR-001 | [Plain English description] | [Validation / Calculation / Eligibility / Routing / Threshold] | [COBOL paragraph name] |

### Business Calculations
| Calc ID | Description | Formula | Example |
|---|---|---|---|
| CALC-001 | [description] | [formula] | [worked example] |

### Exception Handling (Business Perspective)
| Scenario | Business Response |
|---|---|
| [error scenario] | [what the business expects to happen] |

### Reports & Outputs
| Output | Description | Audience | Frequency |
|---|---|---|---|
| [report name] | [what it contains] | [who reads it] | [how often] |

### Compliance & Regulatory Notes
[Any rules that appear regulatory in nature]

### Open Questions for Business SME
[Anything ambiguous — hardcoded values, contradictory logic, unclear rules]
```

### Prompt 2.2 — Business Rules Catalog (Aggregation)

```
ROLE: You are a senior business analyst consolidating business rules across multiple programs into a master catalog.

CONTEXT:
- System: {{SYSTEM_NAME}}
- Domain: {{DOMAIN_CONTEXT}}

TASK: Consolidate business rules from multiple programs. Deduplicate, categorize, flag contradictions, and group related rules.

EXTRACTED RULES FROM MULTIPLE PROGRAMS:
[Paste consolidated output from multiple Prompt 2.1 runs]

OUTPUT FORMAT:

## MASTER BUSINESS RULES CATALOG — {{SYSTEM_NAME}}

### Summary Statistics
- Total Unique Rules: [count]
- Categories: [list with counts]
- Rules in Multiple Programs: [count]
- Potential Contradictions: [count]

### Rules by Category

#### Validation Rules
| Rule ID | Description | Source Program(s) | Priority | Confidence |
|---|---|---|---|---|
| BR-V-001 | [description] | [programs] | [CRITICAL/HIGH/MEDIUM/LOW] | [HIGH/MEDIUM/LOW] |

#### Calculation Rules
| Rule ID | Description | Formula | Source Program(s) | Confidence |
|---|---|---|---|---|
| BR-C-001 | [description] | [formula] | [programs] | [confidence] |

[Continue for Eligibility, Routing, Threshold categories...]

### Cross-Program Rule Instances
| Rule ID | Programs Where Found | Consistent? | Notes |
|---|---|---|---|
| [rule ID] | [list of programs] | [YES/NO/PARTIAL] | [differences] |

### Potential Contradictions
| Rule A | Rule B | Programs | Nature of Contradiction |
|---|---|---|---|
| [rule ID] | [rule ID] | [programs] | [conflict description] |

### Rules Requiring SME Validation
| Rule ID | Reason for Uncertainty |
|---|---|
| [rule ID] | [why this needs human review] |
```

---

## Chapter 11: Pass 3 — Functional Specification Prompts

### Prompt 3.1 — Functional Specification Document

```
ROLE: You are a senior systems analyst writing functional specifications for a legacy modernization project. Your audience is a Java development team who have NEVER seen COBOL and never will. Your specifications must be complete, unambiguous, and platform-independent.

CONTEXT:
- System: {{SYSTEM_NAME}}
- Domain: {{DOMAIN_CONTEXT}}
- Business Process Document (from Pass 2): {{BUSINESS_SPEC}}
- Inventory Entry (from Pass 1): {{PREVIOUS_PASS_OUTPUT}}

TASK: Produce a detailed functional specification for the following COBOL program/module.

SOURCE CODE:
{{COBOL_SOURCE}}

COPYBOOKS:
{{COPYBOOK_SOURCE}}

OUTPUT FORMAT:

## FUNCTIONAL SPECIFICATION — [Module/Function Name]

### Document Control
- **Spec ID:** FS-[unique ID]
- **Source Program(s):** [COBOL program name(s)]
- **Related Business Process:** [Reference]
- **Status:** Draft — Pending SME Review

### 1. Overview
**Purpose:** [2-3 sentences]
**Scope:** [Included and excluded]
**Trigger:** [What initiates this function]

### 2. Input Specification
**Input: [Input Name]**
- **Source:** [API call / database / file / message queue / user input]
- **Format:** [Data format]
- **Fields:**
| Field Name | Data Type | Size/Precision | Required | Validation Rules | Description |
|---|---|---|---|---|---|
| [field name] | [String/Integer/Decimal/Date/Boolean] | [details] | [Y/N] | [rule IDs] | [description] |

### 3. Output Specification
**Output: [Output Name]**
- **Destination:** [API response / database / file / screen / report]
- **Fields:**
| Field Name | Data Type | Size/Precision | Derivation | Description |
|---|---|---|---|---|
| [field name] | [type] | [size] | [How calculated/derived] | [description] |

### 4. Processing Logic
Describe the complete processing flow. Use numbered steps. Do NOT use COBOL terminology.

**4.1 [First Major Processing Block]**
1. [Step]
   - If [condition]: [action]
   - Else: [action]

**4.2 [Next Block]**
[Continue...]

### 5. Validation Rules (Detailed)
| Rule ID | Field(s) | Condition | Error Response | Error Code |
|---|---|---|---|---|
| BR-V-xxx | [fields] | [condition] | [response] | [code] |

### 6. Business Calculations (Detailed)
| Calc ID | Description | Formula | Inputs | Output | Precision | Rounding |
|---|---|---|---|---|---|---|
| BR-C-xxx | [description] | [formula] | [inputs] | [output] | [decimal places] | [rounding rule] |

### 7. State Management
| Current State | Event/Condition | New State | Side Effects |
|---|---|---|---|
| [state] | [trigger] | [new state] | [what else happens] |

### 8. Error Handling
| Error Condition | Detection | Response | Recovery | Notification |
|---|---|---|---|---|
| [condition] | [how detected] | [system response] | [auto-recovery?] | [who gets notified] |

### 9. Integration Points
| System/Service | Direction | Protocol | Data Format | Error Handling | SLA |
|---|---|---|---|---|---|
| [system] | [In/Out/Both] | [REST/MQ/File] | [JSON/XML/Fixed] | [retry, fallback] | [response time] |

### 10. Non-Functional Considerations
- **Expected Volume:** [transactions per day/hour]
- **Performance:** [response time or throughput]
- **Concurrency:** [locking requirements]
- **Idempotency:** [safe to re-run?]
- **Audit:** [what needs logging]

### 11. Traceability
| Business Rule ID | Functional Spec Section | COBOL Source Location |
|---|---|---|
| BR-xxx | Section [N] | [Program, paragraph] |

### 12. Open Items & Assumptions
| Item | Type | Description | Impact if Wrong |
|---|---|---|---|
| [item] | [ASSUMPTION / OPEN QUESTION / RISK] | [description] | [consequence] |
```

### Prompt 3.2 — Screen/UI Mapping Document

```
ROLE: You are a UI/UX analyst documenting legacy mainframe screens for modernization.

CONTEXT:
- System: {{SYSTEM_NAME}}
- Domain: {{DOMAIN_CONTEXT}}

TASK: Analyze the CICS BMS map and COBOL program to document every screen, field, navigation path, and user interaction.

BMS MAP SOURCE:
{{CICS_MAP_SOURCE}}

COBOL PROGRAM:
{{COBOL_SOURCE}}

COPYBOOKS:
{{COPYBOOK_SOURCE}}

OUTPUT FORMAT:

## UI MAPPING DOCUMENT — [Transaction/Screen Name]

### Screen Overview
- **Transaction ID:** [CICS transaction ID]
- **Map Set:** [MAPSET name]
- **Purpose:** [What the user does on this screen]

### Field Inventory
| Field Name | Label | Type | Length | Attributes | Required | Validation | Business Purpose |
|---|---|---|---|---|---|---|---|
| [field] | [label] | [Input/Output/Both] | [chars] | [Protected/Unprotected] | [Y/N] | [rules] | [meaning] |

### User Actions (PF Keys)
| Key | Label | Action | Navigation Target | Preconditions |
|---|---|---|---|---|
| [PF key] | [label] | [what happens] | [next screen] | [requirements] |

### Navigation Flow
- **Entry Points:** [How users arrive]
- **Exit Points:** [Where users can go]
- **Error Flow:** [What happens on validation failure]

### Modern UI Recommendations
[How this screen should be redesigned for web]
```

### Prompt 3.3 — Report & Output Specifications

```
ROLE: You are a reporting analyst documenting legacy report specifications.

CONTEXT:
- System: {{SYSTEM_NAME}}
- Domain: {{DOMAIN_CONTEXT}}

TASK: Document every report generated by the following COBOL program.

SOURCE CODE:
{{COBOL_SOURCE}}

COPYBOOKS:
{{COPYBOOK_SOURCE}}

OUTPUT FORMAT:

## REPORT SPECIFICATION — [Report Name]

### Report Overview
- **Report ID:** RPT-[unique ID]
- **Source Program:** [COBOL program name]
- **Business Purpose:** [Why this report exists]
- **Audience:** [Who reads it]
- **Frequency:** [Daily/Weekly/Monthly/On-Demand]

### Selection Criteria
| Criterion | Condition | Source |
|---|---|---|
| [e.g., "Date Range"] | [condition] | [input parameter or hardcoded] |

### Sort Order
| Level | Field | Direction |
|---|---|---|
| [Primary] | [sort field] | [Ascending/Descending] |

### Report Layout
**Detail Line(s):**
| Column | Field | Format | Width | Alignment | Description |
|---|---|---|---|---|---|
| [column] | [field] | [format] | [chars] | [Left/Right/Center] | [what it shows] |

**Group Breaks:**
| Break Level | Break Field | Subtotal Fields | Other Actions |
|---|---|---|---|
| [level] | [field] | [summed fields] | [page break, etc.] |

**Grand Totals:**
| Total Field | Calculation | Format |
|---|---|---|
| [field] | [SUM/COUNT/AVG] | [format] |

### Modernization Recommendation
[Dashboard, API, CSV/PDF download, or eliminate?]
```

### Prompt 3.4 — Interface & Integration Specification

```
ROLE: You are an integration architect documenting all external interfaces.

CONTEXT:
- System: {{SYSTEM_NAME}}
- Domain: {{DOMAIN_CONTEXT}}

TASK: Document every external interface in the following COBOL program.

SOURCE CODE:
{{COBOL_SOURCE}}

JCL:
{{JCL_SOURCE}}

COPYBOOKS:
{{COPYBOOK_SOURCE}}

OUTPUT FORMAT:

## INTERFACE SPECIFICATION — [Interface Name]

### Interface Overview
- **Interface ID:** INT-[unique ID]
- **Source Program:** [COBOL program name]
- **Direction:** [Inbound / Outbound / Bidirectional]
- **External System:** [Name if known]
- **Protocol:** [File Transfer / MQ / CICS Link / DB2 Shared Table / API]
- **Frequency:** [Real-time / Batch Daily / On-demand]

### Data Format
- **Format Type:** [Fixed-Width / Delimited / XML / JSON / Binary]
- **Character Encoding:** [EBCDIC / ASCII / UTF-8]
- **Record Length:** [fixed or variable]

### Record Layout
| Field | Position | Length | Type | Format | Required | Description |
|---|---|---|---|---|---|---|
| [field] | [start-end] | [bytes] | [Alpha/Numeric/Packed] | [details] | [Y/N] | [meaning] |

### Error Handling
- **File-Level Errors:** [missing / empty / corrupt handling]
- **Record-Level Errors:** [individual record failure handling]
- **Error Reporting:** [how errors are communicated]

### Volume & Performance
- **Typical Volume:** [records per transmission]
- **Peak Volume:** [maximum expected]
- **Processing Window:** [SLA / deadline]

### Modernization Considerations
[REST API, event stream, batch file, or database integration?]
```

---

## Chapter 12: Pass 4 — Technical Specification Prompts

### Prompt 4.1 — Low-Level Technical Specification

```
ROLE: You are a senior technical architect writing detailed technical specifications for Java developers who will reimplement COBOL functionality. Be precise about data types, precision, edge cases, and processing sequences.

CONTEXT:
- System: {{SYSTEM_NAME}}
- Domain: {{DOMAIN_CONTEXT}}
- Functional Specification: {{FUNCTIONAL_SPEC}}
- Business Specification: {{BUSINESS_SPEC}}

TASK: Produce a low-level technical specification for the following COBOL program.

SOURCE CODE:
{{COBOL_SOURCE}}

COPYBOOKS:
{{COPYBOOK_SOURCE}}

OUTPUT FORMAT:

## TECHNICAL SPECIFICATION — {{PROGRAM_NAME}}

### Document Control
- **Tech Spec ID:** TS-[unique ID]
- **Source Program:** {{PROGRAM_NAME}}
- **Related Functional Spec:** FS-[reference]

### 1. Program Overview
- **Type:** [Batch / Online / Subroutine]
- **Entry Point:** [Main section/paragraph]
- **Exit Point(s):** [STOP RUN / GOBACK / CICS RETURN]
- **Return Codes:** [List all and their meanings]

### 2. Data Structures — Java Mapping

**[Structure Name] → Suggested Java Class: [ClassName]**
| COBOL Field | PIC Clause | COBOL Type | Java Type | Java Field Name | Validation | Notes |
|---|---|---|---|---|---|---|
| [field] | [PIC] | [type] | [Java type] | [camelCase] | [annotations] | [conversion notes] |

### 3. Processing Logic — Detailed Pseudocode

Express the COBOL logic as structured pseudocode mapping to Java methods.

```pseudocode
METHOD [methodName]([params]): [returnType]
  
  // BR-V-001: [Rule description]
  IF [condition]:
    THROW ValidationException("[error code]", "[message]")
  
  // BR-C-001: [Calculation description]
  // CRITICAL: Use BigDecimal with HALF_UP rounding, scale=2
  dailyRate = annualRate.divide(BigDecimal.valueOf(360), 10, HALF_UP)
  dailyInterest = balance.multiply(dailyRate).setScale(2, HALF_UP)
  
  RETURN [result]
```

### 4. Critical Precision & Arithmetic Notes
| Concern | COBOL Behavior | Java Equivalent | Risk if Wrong |
|---|---|---|---|
| [e.g., "Decimal arithmetic"] | [COBOL behavior] | [BigDecimal approach] | [consequence] |

### 5. Error Handling Mapping
| COBOL Pattern | COBOL Behavior | Java Equivalent |
|---|---|---|
| FILE STATUS check | Status "35" = file not found | throw FileNotFoundException |
| SQLCODE check | +100 = not found, -xxx = error | Optional.empty() or SQLException |
| CICS RESP check | RESP = NOTFND | throw NotFoundException |
| ABEND | ABEND code Uxxxx | throw RuntimeException with code |
| RETURN-CODE | 0/4/8/12/16 | enum ResultCode mapping |

### 6. Concurrency & Locking
- **File Locking:** [Exclusive / Shared]
- **DB2 Locking:** [Isolation level, FOR UPDATE cursors]
- **CICS Enqueue:** [Resource-level locking]

### 7. Performance Characteristics
- **Current Volume:** [Records per run]
- **Processing Pattern:** [Sequential / Random / Cursor-based]
- **Java Optimization Notes:** [Batch insert, streaming, caching suggestions]

### 8. Traceability Matrix
| Business Rule | Functional Spec Section | COBOL Location | Tech Spec Section | Suggested Java Method |
|---|---|---|---|---|
| BR-xxx | FS section | Program, paragraph | Section above | methodName() |
```

### Prompt 4.2 — Data Model Design Document

```
ROLE: You are a data architect designing a modern relational data model to replace legacy COBOL structures.

CONTEXT:
- System: {{SYSTEM_NAME}}
- Domain: {{DOMAIN_CONTEXT}}

TASK: Design a normalized modern data model from the COBOL data structures.

DATA DICTIONARIES (from Pass 1):
[Paste aggregated data dictionary outputs]

BUSINESS PROCESS DOCUMENTS (from Pass 2):
[Paste relevant summaries]

OUTPUT FORMAT:

## DATA MODEL DESIGN — {{SYSTEM_NAME}}

### 1. Design Principles
[Normalized, domain-driven, event-sourced — justify choice]

### 2. Entity Definitions
For each entity:

**Entity: [Entity Name]**
- **Source:** [COBOL structures this replaces]
- **Suggested Java Class:** [ClassName]
- **Suggested Table Name:** [table_name]

| Column | Java Type | DB Type | Nullable | Default | Constraints | Source Field(s) | Notes |
|---|---|---|---|---|---|---|---|
| [column] | [Java type] | [SQL type] | [Y/N] | [default] | [PK/FK/UNIQUE/CHECK] | [COBOL field(s)] | [notes] |

**Indexes:**
| Index Name | Columns | Type | Rationale |
|---|---|---|---|
| [name] | [columns] | [UNIQUE/NON-UNIQUE] | [based on COBOL access patterns] |

### 3. Data Type Translation Guide
| COBOL Pattern | Example | Java Type | SQL Type | Migration Notes |
|---|---|---|---|---|
| PIC X(n) | PIC X(30) | String | VARCHAR(30) | Trim trailing spaces |
| PIC 9(n) | PIC 9(8) | int or long | INTEGER/BIGINT | Remove leading zeros |
| PIC S9(n)V9(m) COMP-3 | PIC S9(7)V99 COMP-3 | BigDecimal | DECIMAL(9,2) | Preserve exact scale |
| PIC 9(8) date | 20240115 | LocalDate | DATE | Parse YYYYMMDD, 00000000→null |

### 4. Data Migration Considerations
| Source | Target | Transformation | Complexity |
|---|---|---|---|
| [COBOL structure] | [new entity] | [what changes] | [LOW/MEDIUM/HIGH] |

### 5. Eliminated Redundancies
[Data duplications removed in new model]
```

### Prompt 4.3 — Service Decomposition Map

```
ROLE: You are a solution architect designing the target service architecture.

CONTEXT:
- System: {{SYSTEM_NAME}}
- Domain: {{DOMAIN_CONTEXT}}

TASK: Propose how the monolithic COBOL system should be decomposed into modern services.

INPUTS:
- Application Inventory: [from Pass 1]
- Dependency Graph: [from Pass 1]
- Business Process Documents: [from Pass 2]
- Data Model Design: [from Prompt 4.2]

OUTPUT FORMAT:

## SERVICE DECOMPOSITION MAP — {{SYSTEM_NAME}}

### 1. Architecture Pattern
[Microservices / Modular Monolith / Hybrid — justify]

### 2. Bounded Contexts
| Context Name | Business Capability | Key Entities | COBOL Programs Replaced |
|---|---|---|---|
| [name] | [capability] | [entities] | [programs] |

### 3. Service Definitions
**Service: [Service Name]**
- **Bounded Context:** [context]
- **Responsibility:** [single sentence]
- **COBOL Programs Replaced:** [list]
- **API Surface:**
  | Endpoint | Method | Description | Consumers |
  |---|---|---|---|
  | [/api/v1/...] | [GET/POST/PUT/DELETE] | [description] | [callers] |
- **Data Owned:** [entities/tables]
- **Dependencies:** [other services, sync/async]

### 4. Migration Sequence
| Phase | Service(s) | Rationale | Risk |
|---|---|---|---|
| 1 | [services] | [why first] | [risks] |
| 2 | [services] | [why next] | [risks] |

### 5. Strangler Fig Opportunities
[Services that can run alongside mainframe during migration]
```

### Prompt 4.4 — Error Handling & Logging Strategy

```
ROLE: You are a senior Java architect designing error handling and observability.

CONTEXT:
- System: {{SYSTEM_NAME}}
- All COBOL error handling patterns from prior passes

TASK: Map legacy COBOL error patterns to modern Java error handling.

COBOL ERROR PATTERNS:
[Paste consolidated error handling sections]

OUTPUT FORMAT:

## ERROR HANDLING & LOGGING STRATEGY — {{SYSTEM_NAME}}

### 1. Pattern Mapping
| COBOL Pattern | Example | Java Pattern | Notes |
|---|---|---|---|
| Return Code | RC 0/4/8/12/16 | ResultCode enum + exit code | Map: 0=SUCCESS, 4=WARNING, 8+=ERROR |
| FILE STATUS | IF FILE-STATUS NOT = "00" | try/catch IOException | Create FileStatus enum |
| SQLCODE | IF SQLCODE = +100 | Optional.empty() | Map common codes |
| CICS RESP | IF RESP = DFHRESP(NOTFND) | Custom exceptions | Per resource type |
| ABEND | ABEND ABCODE('Uxxxx') | throw SystemException(code) | Global handler |
| ON SIZE ERROR | ON SIZE ERROR PERFORM ... | ArithmeticException | Test boundary values |

### 2. Exception Hierarchy
[Java exception class hierarchy]

### 3. Logging Strategy
| Level | When | COBOL Equivalent | Example |
|---|---|---|---|
| ERROR | Unrecoverable | ABEND, RC=12/16 | "Failed to process account {id}" |
| WARN | Recoverable | RC=4/8 | "Skipped record: invalid date" |
| INFO | Business events | DISPLAY | "Processed {count} records" |
| DEBUG | Technical detail | None | "Query returned {rows} rows" |

### 4. Batch Error Handling
[Continue on error vs. stop, threshold, restart capability]
```

### Prompt 4.5 — Batch Processing Design

```
ROLE: You are a Java architect designing modern batch processing.

CONTEXT:
- System: {{SYSTEM_NAME}}
- Domain: {{DOMAIN_CONTEXT}}

TASK: Design modern batch architecture from JCL analysis and functional specs.

JCL ANALYSIS (from Pass 1):
[Paste outputs]

FUNCTIONAL SPECS (from Pass 3):
[Paste summaries]

OUTPUT FORMAT:

## BATCH PROCESSING DESIGN — {{SYSTEM_NAME}}

### 1. Technology Recommendation
[Spring Batch / custom / workflow orchestrator — justify]

### 2. Job Mapping
**Legacy Job: [JCL Name] → Modern Job: [New Name]**
| Legacy Step | Legacy Program | Modern Component | Processing Pattern |
|---|---|---|---|
| [step] | [COBOL program] | [Java class/service] | [Chunk / Tasklet / Stream] |

### 3. Data Flow Redesign
[Legacy used intermediate files; modern might use in-memory, staging tables, or queues]

### 4. Restart & Recovery
| Job | Checkpoint Strategy | Restart Capability | Idempotency |
|---|---|---|---|
| [job] | [chunk/step/none] | [from checkpoint / from beginning] | [approach] |

### 5. Scalability
| Job | Current Volume | Growth | Scaling Strategy |
|---|---|---|---|
| [job] | [records] | [expected] | [Partitioning / Parallel / Horizontal] |
```

---

## Chapter 13: Pass 5 — Governance & Cross-Cutting Prompts

### Prompt 5.1 — Traceability Matrix (Aggregation)

```
ROLE: You are a QA architect ensuring completeness across the modernization.

CONTEXT:
- System: {{SYSTEM_NAME}}

TASK: Produce a complete traceability matrix from all prior artifacts.

INPUTS:
- Business Rules Catalog (Pass 2)
- Functional Specifications (Pass 3)
- Technical Specifications (Pass 4)

OUTPUT FORMAT (structured table):

## TRACEABILITY MATRIX — {{SYSTEM_NAME}}

| Trace ID | Business Rule ID | Rule Description | Business Process | Functional Spec | Tech Spec | COBOL Source | Target Service | Target Component | Test Scenario | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| TRC-001 | BR-V-001 | [description] | [process] | FS-xxx §N | TS-xxx §N | [program, para] | [service] | [Class.method()] | TC-xxx | COMPLETE |

### Coverage Analysis
- Business Rules with full traceability: [count] / [total] ([%])
- Rules missing functional spec: [list]
- Rules missing technical spec: [list]
- Rules missing test coverage: [list]
```

### Prompt 5.2 — Gap & Risk Register

```
ROLE: You are a modernization risk analyst reviewing all artifacts.

CONTEXT:
- System: {{SYSTEM_NAME}}

TASK: Consolidate all open questions, assumptions, and flags from Passes 1-4 into a prioritized risk register.

ALL OPEN ITEMS:
[Paste all open questions, assumptions, and flags]

OUTPUT FORMAT:

## GAP & RISK REGISTER — {{SYSTEM_NAME}}

### Summary
- Total Items: [count]
- Critical: [count] | High: [count] | Medium: [count] | Low: [count]

### Register
| Risk ID | Category | Description | Source | Affected Components | Severity | Likelihood | Impact | Action | Owner | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| RISK-001 | [AMBIGUOUS LOGIC / DEAD CODE / MISSING KNOWLEDGE / etc.] | [description] | [artifact] | [components] | [CRITICAL/HIGH/MEDIUM/LOW] | [H/M/L] | [consequence] | [mitigation] | [role] | [OPEN] |

### Assumptions Log
| Assumption ID | Description | Impact if Wrong | Validation Method |
|---|---|---|---|
| ASMP-001 | [assumption] | [consequence] | [how to verify] |
```

### Prompt 5.3 — Test Scenario Catalog

```
ROLE: You are a QA architect designing the test strategy for behavioral equivalence.

CONTEXT:
- System: {{SYSTEM_NAME}}
- Domain: {{DOMAIN_CONTEXT}}

TASK: Produce comprehensive test scenarios from functional and technical specs.

FUNCTIONAL SPECS: {{FUNCTIONAL_SPEC}}
BUSINESS RULES: [Paste business rules catalog]

OUTPUT FORMAT:

## TEST SCENARIO CATALOG — [Module/Service Name]

### Testing Approach
- **Strategy:** [Parallel run / Shadow / Record-replay / Unit + Integration]
- **Equivalence Criteria:** [Exact match / tolerance]

#### Happy Path Scenarios
| Scenario ID | Description | Input Conditions | Expected Outcome | Rules Tested | Priority |
|---|---|---|---|---|---|
| TC-HP-001 | [description] | [inputs] | [expected] | [BR-xxx] | [P1/P2/P3] |

#### Boundary & Edge Cases
| Scenario ID | Description | Input Conditions | Expected Outcome | Risk if Missed | Priority |
|---|---|---|---|---|---|
| TC-BC-001 | [e.g., "Maximum balance value"] | [PIC S9(7)V99 max = 9999999.99] | [handling] | [overflow risk] | P1 |

#### Error Scenarios
| Scenario ID | Description | Error Condition | Expected Behavior | COBOL RC | Java Equivalent |
|---|---|---|---|---|---|
| TC-ER-001 | [description] | [condition] | [response] | [RC] | [Exception type] |

#### COBOL-Specific Precision Scenarios
| Scenario ID | Description | COBOL Behavior | Java Risk | Validation Method |
|---|---|---|---|---|
| TC-PR-001 | Packed decimal precision | [COBOL calc] | [BigDecimal mismatch] | [compare to 2 decimals] |

#### Batch-Specific Scenarios
| Scenario ID | Description | Condition | Expected Behavior |
|---|---|---|---|
| TC-BA-001 | Empty input file | Zero records | [graceful completion] |
| TC-BA-002 | Restart after failure | Mid-batch failure | [resumes, no duplicates] |

### Data Requirements
[Test data sources, PII concerns, synthesis needs]
```

---

# PART IV: SPEC-DRIVEN DEVELOPMENT LIBRARY (Machine-Readable Contracts)

---

## Chapter 14: Domain Model Specification (JSON Schema)

### LLM Prompt

```
ROLE: You are a domain modeling specialist producing machine-readable JSON Schema specifications from COBOL copybooks and programs. Your output will be consumed by code generators.

CONTEXT:
- System: {{SYSTEM_NAME}}
- Domain: {{DOMAIN_CONTEXT}}

TASK: Convert the following COBOL data structures into a JSON Schema domain model specification. For EVERY field, include:
1. JSON Schema type and constraints
2. x-cobol-field: original PIC clause, storage type, byte size
3. x-database: target column name, SQL type, constraints
4. x-java: Java type, Bean Validation annotations
5. x-precision: for numeric fields — integer/decimal digits, rounding mode
6. x-migration: transformation needed during data migration
7. x-business-rules: business rule IDs that govern this field

For 88-level condition names, map to enum values.
For REDEFINES, model as separate optional fields with discriminator or union type.

COBOL COPYBOOKS:
{{COPYBOOK_SOURCE}}

COBOL PROGRAMS (for usage context):
{{COBOL_SOURCE}}

OUTPUT: Valid YAML conforming to JSON Schema draft 2020-12 with custom x- extensions. Output ONLY the YAML.
```

### Example Output Structure

See the full example in the Spec-Driven Development document (cobol_sdd_approach.md), Chapter: Artifact 1 — Domain Model Specification, which includes a complete `account.schema.yaml` with all x- extension properties.

---

## Chapter 15: API Contract Specification (OpenAPI 3.1)

### LLM Prompt

```
ROLE: You are an API architect producing OpenAPI 3.1 specifications from COBOL programs.

CONTEXT:
- System: {{SYSTEM_NAME}}
- Domain: {{DOMAIN_CONTEXT}}
- Domain Model Schema (from Chapter 14): {{DOMAIN_MODEL_SPEC}}

TASK: Convert the following COBOL program into an OpenAPI 3.1 specification.

For ONLINE/CICS programs: Each transaction → API endpoint. RECEIVE MAP → request body. SEND MAP → response. PF keys → distinct endpoints.
For BATCH programs: Batch process → POST endpoint. Input files → request body. Return codes → HTTP status codes.
For SUBROUTINES: LINKAGE SECTION → method signature. Decide: internal method or separate API.

For EVERY endpoint include:
1. x-cobol-source: program, paragraph, transaction, JCL job
2. x-business-rules: list of business rule IDs
3. x-precision-critical: flag financial calculations with exact precision specs

COBOL error → HTTP status mapping:
- SQLCODE +100 → 404 | SQLCODE -803 → 409 | SQLCODE < 0 → 500
- FILE STATUS ≠ "00" → appropriate 4xx/5xx
- CICS NOTFND → 404 | CICS DUPRC → 409
- ABEND → 500 | RC 0 → 200 | RC 4 → 200 with warnings | RC 8+ → 500

$ref domain model schemas wherever possible.

COBOL SOURCE:
{{COBOL_SOURCE}}

COPYBOOKS:
{{COPYBOOK_SOURCE}}

OUTPUT: Valid OpenAPI 3.1 YAML only. No prose.
```

---

## Chapter 16: Business Rules Specification (YAML DSL)

### LLM Prompt

```
ROLE: You are a business rules engineer extracting rules from COBOL into a structured YAML DSL. Your output must be machine-parseable and include embedded test cases.

CONTEXT:
- System: {{SYSTEM_NAME}}
- Domain: {{DOMAIN_CONTEXT}}
- Domain Model (from Chapter 14): {{DOMAIN_MODEL_SPEC}}

TASK: Extract ALL business rules from the COBOL program into structured YAML.

For EACH rule include:
1. Unique ID: BR-{V|C|E|T|R|S}-{number}
2. x-cobol-source: program, paragraph, line range
3. Machine-parseable condition block (operators: equals, not_equals, greater_than, less_than, in, not_in, is_null, is_blank, matches, between)
4. Machine-parseable action: reject, calculate, skip, apply_fee, route, transform
5. At least 3 test cases: happy path, boundary, negative
6. For calculations: EXACT precision (scale, rounding, day-count convention)
7. Flag uncertain rules with x-sme-review

Pay special attention to:
- Hardcoded values (magic numbers) — extract as named constants, flag for SME
- 88-level condition names — these ARE business rules
- Nested IF — decompose into individual rules
- EVALUATE/WHEN — each branch is likely a separate rule
- COMPUTE with ROUNDED — document exact rounding behavior

COBOL SOURCE:
{{COBOL_SOURCE}}

COPYBOOKS:
{{COPYBOOK_SOURCE}}

OUTPUT: Valid YAML only. No prose.
```

---

## Chapter 17: Behavior Specification (Gherkin/Cucumber)

### LLM Prompt

```
ROLE: You are a BDD test engineer writing Gherkin feature files from COBOL specifications. Features will be executed by Cucumber against Java code.

CONTEXT:
- System: {{SYSTEM_NAME}}
- Domain: {{DOMAIN_CONTEXT}}
- Business Rules (from Chapter 16): {{BUSINESS_RULES_SPEC}}
- API Contract (from Chapter 15): {{API_SPEC}}

TASK: Produce Gherkin feature files. Every business rule must have at least one scenario. Every API endpoint must have success, validation failure, and error scenarios.

REQUIREMENTS:
1. Tag every scenario: @domain:{context} @source:{program} @priority:{P1/P2/P3} @rule:{BR-xxx}
2. Use Scenario Outlines with Examples for parameterized tests (include boundary values: zero, max PIC size, negative, null/blank)
3. For financial calculations, specify EXACT expected values to 2 decimal places
4. Include at least one @parallel-run scenario per feature
5. For batch: empty input, single record, normal volume, mixed errors, restart scenarios

COBOL SOURCE:
{{COBOL_SOURCE}}

BUSINESS RULES SPEC:
{{BUSINESS_RULES_SPEC}}

OUTPUT: Valid Gherkin (.feature) files only. No prose.
```

---

## Chapter 18: Process & Workflow Specification

### LLM Prompt

```
ROLE: You are a workflow architect extracting business processes from COBOL into machine-readable state machine definitions.

CONTEXT:
- System: {{SYSTEM_NAME}}
- Domain: {{DOMAIN_CONTEXT}}
- Business Rules (from Chapter 16): {{BUSINESS_RULES_SPEC}}
- Inventory (from Pass 1): {{INVENTORY}}

TASK: Extract the workflow as a state machine with:
1. States: Each distinct status/phase. Map 88-level condition names.
2. Transitions: Events causing state changes. Map IF/EVALUATE status logic.
3. Guards: Conditions for transitions. Reference business rule IDs.
4. Actions: On entry/exit/during. Map COBOL paragraphs.
5. Scheduled Actions: Batch processes running on schedule per state.

Include x-cobol-source for every state, transition, and action.

COBOL PROGRAMS:
{{COBOL_SOURCE}}

OUTPUT: Valid YAML. No prose.
```

---

## Chapter 19: Event Specification (AsyncAPI 3.0)

### LLM Prompt

```
ROLE: You are an event-driven architecture specialist extracting asynchronous event contracts from COBOL programs.

TASK: Analyze the COBOL program for data published or sent to external systems (MQ PUT, output files for other systems, CICS START, TD queue writes). Convert each into an AsyncAPI 3.0 channel definition.

For each event:
1. Channel name: domain/entity/event-name
2. Complete payload schema with types and precision
3. x-cobol-source: original program and mechanism
4. When triggered

COBOL SOURCE:
{{COBOL_SOURCE}}

OUTPUT: Valid AsyncAPI 3.0 YAML only.
```

---

## Chapter 20: Data Migration Specification

### LLM Prompt

```
ROLE: You are a data migration specialist creating machine-readable mapping specs.

CONTEXT:
- System: {{SYSTEM_NAME}}
- Domain Model (from Chapter 14): {{DOMAIN_MODEL_SPEC}}
- Data Dictionary (from Pass 1): [Paste data dictionary]

TASK: Create migration mapping for each data store.

For EACH field:
1. Source: exact COBOL/DB2 column, type, size
2. Target: new column, type, size
3. Transform: DIRECT, TRIM, MAP, DATE_PARSE, DECIMAL_CONVERT, CUSTOM
4. For MAP: complete value mapping including spaces/nulls/unknowns
5. For DATE_PARSE: format, null representations, invalid date handling
6. Precision warnings for decimal fields
7. Validation rules (pre and post migration): row counts, financial totals, sample comparison

COBOL COPYBOOKS:
{{COPYBOOK_SOURCE}}

DB2 DDL (if available):
{{DB2_DDL}}

OUTPUT: Valid YAML only. No prose.
```

---

# PART V: UNIFIED DUAL-FORMAT LIBRARY (Recommended)

---

## Chapter 21: Pass 1 — Dual-Format Discovery Prompts

### Prompt DF-1.1 — Inventory & Dependency Map

```
ROLE: You are a COBOL mainframe analyst. You produce output in TWO formats: human-readable summary for stakeholders, and machine-readable YAML for automation.

CONTEXT:
- System: {{SYSTEM_NAME}}
- Domain: {{DOMAIN_CONTEXT}}

TASK: Analyze the following COBOL program and produce a dual-format inventory entry.

SOURCE CODE:
{{COBOL_SOURCE}}

COPYBOOKS:
{{COPYBOOK_SOURCE}}

JCL (if available):
{{JCL_SOURCE}}

===== OUTPUT FORMAT =====

Produce TWO sections separated by the marker "---MACHINE-READABLE-SPEC---"

## SECTION 1: HUMAN-READABLE INVENTORY

### Program Summary: [Program Name]

**What This Program Does:**
[2-3 sentences in plain business English. No COBOL jargon.]

**Business Process:** [Which process this belongs to]
**Program Type:** [Batch / Online / Subroutine / Utility]
**Complexity:** [Low / Medium / High / Very High] — [rationale]

**What It Depends On:**
[Narrative describing calls, data, databases, external systems]

**What Depends On It:**
[If determinable]

**Key Business Rules Found:**
[List important rules with preliminary BR-xxx IDs]

**Flags for Human Review:**
[Dead code, commented sections, hardcoded values, possible bugs]

**Data Stores Touched:**
| Data Store | Type | Access | Brief Purpose |
|---|---|---|---|
| [name] | [DB2/VSAM/Seq/MQ] | [Read/Write/Both] | [purpose] |

---MACHINE-READABLE-SPEC---

## SECTION 2: MACHINE-READABLE INVENTORY

```yaml
# inventory/{{PROGRAM_NAME}}.inventory.yaml
program:
  id: "[PROGRAM-ID]"
  type: "[BATCH | ONLINE_CICS | SUBROUTINE | UTILITY]"
  lines_of_code: [count]
  complexity: "[LOW | MEDIUM | HIGH | VERY_HIGH]"
  summary: "[one-line description]"
  business_process: "[process name]"
  domain: "[bounded context]"

  dependencies:
    calls_programs:
      - program: "[name]"
        call_type: "[STATIC | DYNAMIC | CICS_LINK | CICS_XCTL]"
        parameters: [list]
        location: "[paragraph]"
    copybooks:
      - name: "[name]"
        division: "[DATA | PROCEDURE | BOTH]"
        purpose: "[description]"
    files:
      - fd_name: "[name]"
        dd_name: "[name]"
        access_mode: "[SEQUENTIAL | RANDOM | DYNAMIC]"
        operations: ["READ", "WRITE"]
    database:
      - table: "[name]"
        operation: "[SELECT | INSERT | UPDATE | DELETE]"
        key_columns: [list]
    cics_resources:
      - type: "[MAP | TDQ | TSQ]"
        name: "[name]"
        operation: "[SEND | RECEIVE]"

  data_flow:
    inputs:
      - source: "[description]"
        type: "[FILE | DB2 | SCREEN | MQ]"
    outputs:
      - target: "[description]"
        type: "[FILE | DB2 | SCREEN | REPORT]"

  preliminary_business_rules:
    - id: "BR-[category]-[number]"
      summary: "[description]"
      location: "[paragraph]"
      confidence: "[HIGH | MEDIUM | LOW]"

  flags:
    - type: "[DEAD_CODE | UNCLEAR_LOGIC | HARDCODED_VALUE | POSSIBLE_BUG]"
      description: "[what was found]"
      location: "[where]"
      action_needed: "[INVESTIGATE | SME_REVIEW | DOCUMENT]"
```
```

### Prompt DF-1.2 — Data Dictionary & Domain Model

```
ROLE: You are a COBOL data analyst producing a dual-format data dictionary: human-readable for analysts and a JSON Schema domain model for code generators.

CONTEXT:
- System: {{SYSTEM_NAME}}
- Domain: {{DOMAIN_CONTEXT}}

TASK: Extract complete data dictionary from COBOL source and copybooks.

SOURCE CODE:
{{COBOL_SOURCE}}

COPYBOOKS:
{{COPYBOOK_SOURCE}}

===== OUTPUT FORMAT =====

Produce TWO sections separated by "---MACHINE-READABLE-SPEC---"

## SECTION 1: HUMAN-READABLE DATA DICTIONARY

### Data Dictionary: {{PROGRAM_NAME}}

**Overview:**
[2-3 sentences describing major data structures and their business purpose]

#### Record Layouts
**[Record Name] — [Business Purpose]**
[Paragraph describing what this record represents]

| Field | Business Name | Description | Type | Example Values | Notes |
|---|---|---|---|---|---|
| [COBOL field] | [human name] | [meaning] | [e.g., "Dollar amount (2 decimals)"] | [examples] | [special handling] |

#### Key Data Relationships
[How structures relate — foreign keys, linked records]

#### COBOL-Specific Data Patterns
[REDEFINES, packed decimals, condition names, OCCURS, sign handling]

#### Data Quality Observations
[Spaces vs nulls, sentinel dates, overflow risks]

---MACHINE-READABLE-SPEC---

## SECTION 2: MACHINE-READABLE DOMAIN MODEL

```yaml
# domain-model/[entity-name].schema.yaml
$schema: "https://json-schema.org/draft/2020-12/schema"
$id: "https://{{SYSTEM_NAME}}/schemas/[entity].yaml"
title: "[EntityName]"
description: "[Business description]"

x-cobol-source:
  copybook: "[copybook name]"
  programs: ["list"]

x-database:
  table_name: "[table name]"
  schema: "[schema name]"

type: object
required: [list]

properties:
  [fieldName]:
    type: "[string | number | integer | boolean]"
    format: "[date | decimal | etc.]"
    description: "[business description]"
    x-cobol-field:
      name: "[COBOL field name]"
      pic: "[PIC clause]"
      storage: "[alphanumeric | packed-decimal | etc.]"
      bytes: [size]
      condition_names:
        "[value]": "[condition name]"
    x-database:
      column: "[column name]"
      type: "[SQL type]"
      constraints: ["PK", "FK:table.col", "NOT NULL"]
    x-java:
      type: "[Java type]"
      validation: "[Bean Validation]"
    x-precision:
      integer_digits: [N]
      decimal_digits: [N]
      rounding: "[HALF_UP | TRUNCATE]"
    x-migration:
      transform: "[DIRECT | TRIM | MAP | DATE_PARSE]"
      null_representations: ["00000000", " "]
    x-business-rules: ["BR-xxx"]
```
```

---

## Chapter 22: Pass 2 — Dual-Format Business Specification Prompts

### Prompt DF-2.1 — Business Process & Workflow + Rules

```
ROLE: You are BOTH a senior business analyst AND a workflow architect. Produce TWO outputs: human-readable business process document and machine-readable workflow + rules YAML.

CONTEXT:
- System: {{SYSTEM_NAME}}
- Domain: {{DOMAIN_CONTEXT}}
- Inventory (from Pass 1): {{PREVIOUS_PASS_OUTPUT}}

SOURCE CODE:
{{COBOL_SOURCE}}

COPYBOOKS:
{{COPYBOOK_SOURCE}}

===== OUTPUT FORMAT =====

Produce TWO sections separated by "---MACHINE-READABLE-SPEC---"

## SECTION 1: HUMAN-READABLE BUSINESS PROCESS

### Business Process: [Business-Friendly Name]

**Business Objective:**
[2-3 sentences for an executive or product owner]

**Business Context:**
[Where this fits in the larger business]

**Process Narrative:**
1. [Step in plain English]
2. [Next step]
3. [Continue...]

**Business Rules Discovered:**
| Rule ID | Plain English Description | Category | Confidence | Question for SME |
|---|---|---|---|---|
| BR-C-001 | [description] | [category] | [H/M/L] | [question or None] |

**Business Calculations:**
| Calc ID | What It Calculates | Formula | Example | Notes |
|---|---|---|---|---|
| BR-C-001 | [description] | [formula] | [worked example] | [conventions] |

**Actors & Stakeholders:**
| Who | Role | What They Care About |
|---|---|---|
| [actor] | [role] | [interest] |

**Exception Handling (Business View):**
| What Goes Wrong | What Happens | Business Impact |
|---|---|---|
| [scenario] | [response] | [consequence] |

**Reports & Outputs:**
| Output | Who Gets It | What It Contains | How Often |
|---|---|---|---|
| [report] | [audience] | [description] | [frequency] |

**Open Questions for Business Review:**
[Specific questions for SMEs]

---MACHINE-READABLE-SPEC---

## SECTION 2: MACHINE-READABLE WORKFLOW & RULES

### Part A: Workflow Specification

```yaml
# workflows/[process-name].workflow.yaml
workflow:
  id: "WF-[id]"
  version: "1.0.0"
  description: "[description]"
  x-cobol-source:
    programs: ["list"]
    jcl_jobs: ["list"]
  x-human-readable-doc: "[Section 1 reference]"

  trigger:
    type: "[SCHEDULED | EVENT | USER_ACTION]"
    schedule: "[cron or description]"

  states:
    [STATE_NAME]:
      description: "[business meaning]"
      x-cobol-source:
        program: "[program]"
        paragraph: "[paragraph]"
      on_entry:
        - action: "[action]"
          rules: ["BR-xxx"]
      transitions:
        - event: "[EVENT]"
          target: "[STATE]"
          guard:
            rules: ["BR-xxx"]
            condition: "[readable condition]"
          action: "[action during transition]"
      scheduled_actions:
        - action: "[action]"
          schedule: "[frequency]"
          rules: ["BR-xxx"]
```

### Part B: Business Rules Specification

```yaml
# rules/[domain]/[process].rules.yaml
rule_set:
  id: "[id]"
  version: "1.0.0"
  domain: "[domain]"
  x-cobol-source:
    programs: ["list"]

rules:
  - id: "BR-[category]-[number]"
    name: "[name]"
    category: "[VALIDATION | CALCULATION | ELIGIBILITY | THRESHOLD | STATE_TRANSITION | ROUTING]"
    priority: [1-10]
    description: "[plain English]"
    x-cobol-source:
      program: "[program]"
      paragraph: "[paragraph]"
    condition:
      all:  # or any:
        - field: "[field]"
          operator: "[equals | not_equals | greater_than | less_than | in | is_null | matches | between]"
          value: "[value]"
    action:
      type: "[validate | calculate | reject | skip | apply_fee | route]"
      error_code: "[code]"
      error_message: "[message]"
      http_status: [code]
      formula: "[for calculations]"
      target_field: "[output field]"
      precision:
        scale: [N]
        rounding_mode: "[HALF_UP | TRUNCATE]"
    test_cases:
      - description: "[what this tests]"
        input: { field1: "value1" }
        expected: { result: "expected" }
      - description: "[boundary]"
        input: { field1: "boundary_value" }
        expected: { result: "expected" }
      - description: "[negative]"
        input: { field1: "invalid" }
        expected: { valid: false, error_code: "ERR-xxx" }
    x-sme-review:
      needed: [true | false]
      question: "[question]"
      confidence: "[HIGH | MEDIUM | LOW]"
```
```

---

## Chapter 23: Pass 3 — Dual-Format Functional & API Specification Prompts

### Prompt DF-3.1 — Functional Spec & API Contract

```
ROLE: You are BOTH a senior systems analyst (writing for Java developers) AND an API architect (producing OpenAPI specs). Produce dual-format output.

CONTEXT:
- System: {{SYSTEM_NAME}}
- Domain: {{DOMAIN_CONTEXT}}
- Business Process Doc (Pass 2, Section 1): {{BUSINESS_SPEC}}
- Business Rules Spec (Pass 2, Section 2): {{BUSINESS_RULES_SPEC}}
- Domain Model Schema (Pass 1, Section 2): {{DOMAIN_MODEL_SPEC}}
- Inventory (Pass 1): {{INVENTORY}}

SOURCE CODE:
{{COBOL_SOURCE}}

COPYBOOKS:
{{COPYBOOK_SOURCE}}

===== OUTPUT FORMAT =====

Produce TWO sections separated by "---MACHINE-READABLE-SPEC---"

## SECTION 1: HUMAN-READABLE FUNCTIONAL SPECIFICATION

### Functional Specification: [Function/Module Name]
**Spec ID:** FS-[id]  |  **Source:** [program(s)]  |  **Status:** Draft

#### 1. Purpose
[2-3 sentences for Java developers who've never seen COBOL]

#### 2. Inputs
**[Input Name]** — [source]
| Field | Type | Required | Rules | Description |
|---|---|---|---|---|
| [field] | [type] | [Y/N] | [BR-xxx] | [meaning] |

#### 3. Outputs
**[Output Name]** — [destination]
| Field | Type | Derivation | Description |
|---|---|---|---|
| [field] | [type] | [how derived] | [meaning] |

#### 4. Processing Logic
**4.1 [Block Name]** (Source: COBOL paragraph [name])
1. [Step]
   - If [condition]: [action]
   - Else: [alternative]

#### 5. Validation Rules
| Rule ID | Field(s) | Condition | Error Response | HTTP Status |
|---|---|---|---|---|
| BR-V-xxx | [fields] | [condition] | [response] | [status] |

#### 6. Calculations
| Calc ID | Formula | Example | Precision | Rounding | COBOL Source |
|---|---|---|---|---|---|
| BR-C-xxx | [formula] | [example] | [decimals] | [mode] | [location] |

#### 7. Error Handling
| Condition | COBOL Behavior | Java Approach |
|---|---|---|
| [condition] | [COBOL response] | [Java approach] |

#### 8. Non-Functional
- Volume: [N] | Performance: [target] | Concurrency: [needs] | Idempotent: [Y/N]

#### 9. Traceability
| Business Rule | Spec Section | COBOL Location |
|---|---|---|
| BR-xxx | §N | [program, paragraph] |

#### 10. Open Items
| Item | Type | Description | Impact |
|---|---|---|---|
| [item] | [ASSUMPTION / QUESTION / RISK] | [description] | [consequence] |

---MACHINE-READABLE-SPEC---

## SECTION 2: MACHINE-READABLE SPECS

### Part A: API Contract (OpenAPI 3.1)

```yaml
# api-specs/[service].openapi.yaml
openapi: "3.1.0"
info:
  title: "[Service]"
  version: "1.0.0"
  x-cobol-source:
    programs: ["list"]
  x-human-readable-doc: "FS-[id]"

paths:
  /[path]:
    [method]:
      operationId: "[name]"
      summary: "[description]"
      x-cobol-source:
        program: "[program]"
        paragraph: "[paragraph]"
      x-business-rules: ["BR-xxx"]
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
            condition: "SQLCODE = +100"
        "422":
          description: "Business rule violation"
        "500":
          description: "Internal error"
          x-cobol-source:
            condition: "SQLCODE < 0 | ABEND"

components:
  schemas:
    [SchemaName]:
      type: object
      properties:
        [field]:
          $ref: "[domain-model.schema.yaml#/properties/field]"
```

### Part B: Interface Spec (if file/MQ interfaces exist)

```yaml
# interfaces/[name].interface.yaml
interface:
  id: "INT-[N]"
  direction: "[INBOUND | OUTBOUND]"
  x-cobol-source:
    program: "[program]"
  current_protocol:
    type: "[FILE | MQ | CICS_LINK]"
    format: "[FIXED_WIDTH | DELIMITED]"
    encoding: "[EBCDIC | ASCII]"
  proposed_protocol:
    type: "[REST_API | EVENT_STREAM | BATCH_FILE]"
  record_layout:
    - field: "[name]"
      position: "[start-end]"
      length: [N]
      type: "[Alpha | Numeric | Packed]"
      x-domain-model-ref: "[schema.yaml#/properties/field]"
```

### Part C: Event Spec (if async interactions found)

```yaml
# events/[domain]-events.asyncapi.yaml
asyncapi: "3.0.0"
info:
  title: "[Domain] Events"
  version: "1.0.0"
  x-cobol-source:
    programs: ["list"]

channels:
  [domain]/[entity]/[event]:
    x-cobol-source:
      program: "[program]"
      original_mechanism: "[MQ PUT / file write]"
    messages:
      [eventName]:
        payload:
          type: object
          properties:
            [field]:
              $ref: "[domain-model ref]"
```
```

---

## Chapter 24: Pass 4 — Dual-Format Technical & Behavior Specification Prompts

### Prompt DF-4.1 — Technical Spec & Executable Tests

```
ROLE: You are BOTH a senior technical architect (writing Java implementation specs) AND a BDD test engineer (writing Gherkin features). Produce dual-format output.

CONTEXT:
- System: {{SYSTEM_NAME}}
- Domain: {{DOMAIN_CONTEXT}}
- Functional Spec (Section 1): {{FUNCTIONAL_SPEC}}
- API Contract (Section 2): {{API_SPEC}}
- Business Rules (YAML): {{BUSINESS_RULES_SPEC}}
- Domain Model: {{DOMAIN_MODEL_SPEC}}

SOURCE CODE:
{{COBOL_SOURCE}}

COPYBOOKS:
{{COPYBOOK_SOURCE}}

===== OUTPUT FORMAT =====

Produce TWO sections separated by "---MACHINE-READABLE-SPEC---"

## SECTION 1: HUMAN-READABLE TECHNICAL SPECIFICATION

### Technical Specification: {{PROGRAM_NAME}}
**Tech Spec ID:** TS-{{PROGRAM_NAME}}
**Related Specs:** FS-[ref], [openapi-file]

#### 1. Architecture Context
[Where this fits in target architecture. Service, layer, dependencies.]

#### 2. Data Structure Mapping
**[COBOL Structure] → Java Class: [Name]**
| COBOL Field | PIC | Java Type | Java Field | Validation | Migration Notes |
|---|---|---|---|---|---|
| [field] | [PIC] | [type] | [name] | [annotations] | [concerns] |

⚠️ **Precision Rules:** [Every field where COBOL arithmetic differs from Java]

#### 3. Processing Logic (Pseudocode)
```pseudocode
METHOD [name]([params]): [return]
  // BR-V-001: [description]
  IF [condition]:
    THROW ValidationException("[code]", "[message]")
  
  // BR-C-001: [description]
  // CRITICAL: BigDecimal, HALF_UP, scale=2
  result = balance.multiply(rate.divide(360, 10, HALF_UP)).setScale(2, HALF_UP)
  RETURN result
```

#### 4. Error Handling
| COBOL Pattern | Example | Java Implementation |
|---|---|---|
| [pattern] | [code] | [approach] |

#### 5. Performance & Concurrency
[Threading, locking, batch chunking, connection pooling]

#### 6. Batch Design (if applicable)
[Spring Batch structure, reader/processor/writer, chunk size, restart]

#### 7. Traceability
| Rule | Functional Spec | API Endpoint | COBOL Source | Java Component | Test |
|---|---|---|---|---|---|
| BR-xxx | FS-xxx §N | [endpoint] | [program, para] | [Class.method()] | TC-xxx |

---MACHINE-READABLE-SPEC---

## SECTION 2: EXECUTABLE SPECS & MIGRATION

### Part A: Gherkin Behavior Specifications

```gherkin
# features/[domain]/[feature].feature

@domain:[context] @source:{{PROGRAM_NAME}} @priority:P1
Feature: [Feature Name]
  As a [actor]
  I need to [capability]
  So that [business value]

  Background:
    Given [common setup]

  @rule:BR-xxx @precision-critical
  Scenario Outline: [Description]
    Given [precondition with "<variable>"]
    When [action]
    Then [outcome with "<expected>"]

    Examples:
      | variable | expected |
      | [value] | [result] |
      | [boundary] | [result] |
      | [edge] | [result] |

  @rule:BR-xxx @negative @error
  Scenario: [Error scenario]
    Given [error precondition]
    When [action]
    Then the system should respond with status [code]
    And the error code should be "[ERR-xxx]"

  @parallel-run @batch
  Scenario: Parallel run validation
    Given a batch input matching COBOL test data:
      | field1 | field2 |
      | [val] | [val] |
    When the [process] runs
    Then the output should match COBOL:
      | output1 | output2 |
      | [val] | [val] |
```

### Part B: Data Migration Mapping

```yaml
# migration/[entity]-migration.mapping.yaml
migration:
  id: "MIG-[entity]"
  x-human-readable-doc: "TS-{{PROGRAM_NAME}}, Section 2"
  source:
    type: "[DB2 | VSAM | SEQUENTIAL]"
    name: "[name]"
    x-cobol-copybook: "[copybook]"
    estimated_rows: [N]
  target:
    type: "[PostgreSQL | MySQL]"
    table: "[schema.table]"
    schema_ref: "[domain-model.yaml]"

  field_mappings:
    - source:
        column: "[COBOL field]"
        type: "[COBOL type]"
        pic: "[PIC clause]"
      target:
        column: "[new column]"
        type: "[SQL type]"
      transform: "[DIRECT | TRIM | MAP | DATE_PARSE | DECIMAL_CONVERT]"
      mapping:
        "[source_val]": "[target_val]"
      null_representations: ["00000000", " "]
      precision:
        source_scale: [N]
        target_scale: [N]
        rounding: "[mode]"
      on_failure: "[REJECT_RECORD | SET_NULL_AND_LOG | DEFAULT_VALUE]"

  validation_checks:
    pre_migration:
      - check: "ROW_COUNT"
      - check: "BALANCE_SUM"
        tolerance: "0.00"
    post_migration:
      - check: "ROW_COUNT_MATCH"
      - check: "SUM_MATCH"
      - check: "SAMPLE_COMPARE"
        sample_size: "1%"
```
```

---

## Chapter 25: Pass 5 — Dual-Format Governance Prompts

### Prompt DF-5.1 — Governance Report & Machine-Readable Traceability

```
ROLE: You are a QA architect producing dual-format governance output: executive summary for leadership and machine-readable data for tracking tools.

CONTEXT:
- System: {{SYSTEM_NAME}}

TASK: Using ALL prior artifacts, produce traceability and risk analysis.

INPUTS: All outputs from Passes 1-4 (both sections)

===== OUTPUT FORMAT =====

Produce TWO sections separated by "---MACHINE-READABLE-SPEC---"

## SECTION 1: HUMAN-READABLE GOVERNANCE REPORT

### Modernization Governance Report: {{SYSTEM_NAME}}

#### Executive Summary
[2-3 paragraphs: total scope, coverage percentage, critical risks, next steps. Written for CTO.]

#### Coverage Analysis
- Programs Analyzed: [N] of [total] ([%])
- Business Rules Extracted: [N]
  - Full traceability: [N] ([%])
  - Missing functional spec: [N]
  - Missing test coverage: [N]
- API Endpoints Defined: [N]
- Gherkin Scenarios: [N]
- Data Entities Modeled: [N]
- Migration Mappings: [N] of [total]

#### Top Risks Requiring Immediate Attention

**RISK-001: [Title]** (CRITICAL)
[Narrative: what, where, impact, recommended action]

**RISK-002: [Title]** (HIGH)
[Continue for top 10...]

#### Assumptions Made
| # | Assumption | Impact if Wrong | How to Verify |
|---|---|---|---|
| 1 | [assumption] | [consequence] | [method] |

#### Recommended Validation Steps
1. [Most critical]
2. [Next]
3. [Continue...]

---MACHINE-READABLE-SPEC---

## SECTION 2: MACHINE-READABLE GOVERNANCE DATA

### Part A: Traceability Matrix

```yaml
# governance/traceability-matrix.yaml
traceability:
  system: "{{SYSTEM_NAME}}"
  coverage_summary:
    total_business_rules: [N]
    rules_with_full_trace: [N]
    coverage_percentage: [N]

  entries:
    - trace_id: "TRC-001"
      business_rule: "BR-V-001"
      rule_description: "[description]"
      business_process_doc: "[ref]"
      workflow_spec: "WF-[id]"
      functional_spec: "FS-[id]"
      api_spec: "[openapi file]"
      api_endpoint: "[path and method]"
      technical_spec: "TS-[id]"
      cobol_program: "[program]"
      cobol_location: "[paragraph]"
      domain_model: "[schema file]"
      target_service: "[service]"
      target_component: "[Class.method()]"
      gherkin_feature: "[feature file]"
      test_scenarios: ["TC-xxx"]
      migration_mapping: "MIG-[entity]"
      status: "[COMPLETE | PARTIAL | MISSING_SPEC | MISSING_TEST]"
```

### Part B: Risk Register

```yaml
# governance/risk-register.yaml
risk_register:
  system: "{{SYSTEM_NAME}}"
  summary:
    total: [N]
    critical: [N]
    high: [N]
    medium: [N]
    low: [N]

  risks:
    - id: "RISK-001"
      title: "[title]"
      category: "[AMBIGUOUS_LOGIC | DEAD_CODE | MISSING_KNOWLEDGE | DATA_QUALITY | PRECISION | COMPLIANCE]"
      severity: "[CRITICAL | HIGH | MEDIUM | LOW]"
      likelihood: "[HIGH | MEDIUM | LOW]"
      description: "[detail]"
      source_artifact: "[artifact]"
      affected_components:
        programs: ["list"]
        services: ["list"]
        business_rules: ["BR-xxx"]
      impact: "[consequence]"
      recommended_action: "[mitigation]"
      owner_role: "[SME | Architect | Developer]"
      status: "[OPEN | IN_PROGRESS | RESOLVED]"

  assumptions:
    - id: "ASMP-001"
      description: "[assumption]"
      impact_if_wrong: "[consequence]"
      validation_method: "[how to verify]"
      status: "[UNVALIDATED | VALIDATED | INVALIDATED]"
```
```

---

# PART VI: EXECUTION

---

## Chapter 26: Automation Pipeline

### Pipeline Architecture

```
COBOL SOURCE REPO
       │
    SCANNER ──→ Lists programs, resolves COPY dependencies
       │
    ASSEMBLER ──→ For each program: program + copybooks + JCL + context
       │
    LLM API ──→ Sends prompt, receives dual-format response
       │
    SPLITTER ──→ Splits at ---MACHINE-READABLE-SPEC--- marker
       │
  ┌────┴────┐
  │         │
/docs/    /specs/
prose     YAML/OpenAPI/Gherkin
  │         │
Confluence  Git repo &
SharePoint  CI/CD pipeline
```

### Splitter Script (Python)

```python
"""
Splits dual-format LLM output into human-readable and machine-readable files.
"""
import os, sys, re, yaml

SEPARATOR = "---MACHINE-READABLE-SPEC---"

def split_output(llm_output, program_name, pass_number, output_dir):
    if SEPARATOR not in llm_output:
        print(f"WARNING: No separator found for {program_name}")
        save_human(llm_output, program_name, pass_number, output_dir)
        return

    parts = llm_output.split(SEPARATOR, 1)
    save_human(parts[0].strip(), program_name, pass_number, output_dir)
    save_machine(parts[1].strip(), program_name, pass_number, output_dir)

def save_human(content, program_name, pass_number, output_dir):
    human_dir = os.path.join(output_dir, "docs", "prose", f"pass{pass_number}")
    os.makedirs(human_dir, exist_ok=True)
    filepath = os.path.join(human_dir, f"{program_name}.md")
    with open(filepath, 'w') as f:
        f.write(content)
    print(f"Human-readable: {filepath}")

def save_machine(content, program_name, pass_number, output_dir):
    yaml_blocks = re.findall(r'```ya?ml\n(.*?)```', content, re.DOTALL)
    gherkin_blocks = re.findall(r'```gherkin\n(.*?)```', content, re.DOTALL)
    spec_dir = os.path.join(output_dir, "specs")

    for i, block in enumerate(yaml_blocks):
        first_line = block.strip().split('\n')[0]
        if first_line.startswith('#') and '/' in first_line:
            filepath = os.path.join(spec_dir, first_line.lstrip('# ').strip())
        else:
            filepath = os.path.join(spec_dir, f"pass{pass_number}", f"{program_name}_{i}.yaml")
        os.makedirs(os.path.dirname(filepath), exist_ok=True)
        with open(filepath, 'w') as f:
            f.write(block)
        try:
            yaml.safe_load(block)
        except yaml.YAMLError as e:
            print(f"WARNING: Invalid YAML in {filepath}: {e}")

    for i, block in enumerate(gherkin_blocks):
        first_line = block.strip().split('\n')[0]
        if first_line.startswith('#') and '/' in first_line:
            filepath = os.path.join(spec_dir, first_line.lstrip('# ').strip())
        else:
            filepath = os.path.join(spec_dir, "features", f"{program_name}_{i}.feature")
        os.makedirs(os.path.dirname(filepath), exist_ok=True)
        with open(filepath, 'w') as f:
            f.write(block)

if __name__ == "__main__":
    with open(sys.argv[1]) as f:
        content = f.read()
    split_output(content, sys.argv[2], sys.argv[3], sys.argv[4])
```

### Context Window Management

- If program + copybooks exceed context window, split at section/paragraph boundaries
- ALWAYS include DATA DIVISION (or relevant copybooks) even when splitting
- For split programs, summarize previously analyzed sections: "Sections 0000-2999 handle input validation. Section 3000+ (below) handles calculation."
- Feed context forward: when processing Program B called by Program A, include A's specification summary

---

## Chapter 27: Quality Gates & Checkpoints

```yaml
gates:
  after_pass_1:
    human_check:
      - "Program count matches source inventory"
      - "Every program has both human and machine output"
    machine_check:
      - "All inventory YAML syntactically valid"
      - "All domain model schemas conform to JSON Schema"
      - "All x-cobol-field entries have pic and storage"
    sme_review: "10-15% sample of inventory summaries"

  after_pass_2:
    human_check:
      - "Every business process has a narrative document"
      - "Business rule IDs unique, no duplicates"
    machine_check:
      - "All workflow YAML syntactically valid"
      - "All rules have at least 3 test cases"
      - "State machine transitions consistent"
    sme_review: "Business SME validates 15-20% of process docs"

  after_pass_3:
    human_check:
      - "Every program has a functional specification"
      - "All calculations have worked examples"
    machine_check:
      - "All OpenAPI specs pass validation"
      - "All $ref references resolve"
      - "All BR-xxx references exist in rules YAML"
    sme_review: "COBOL developer validates 10-15% of specs"

  after_pass_4:
    human_check:
      - "Every program has a technical specification"
      - "All precision warnings explicit"
    machine_check:
      - "All Gherkin features parse"
      - "Every BR-xxx has at least one scenario"
      - "Migration YAML validation checks defined"
    sme_review: "Java architect reviews data model and decomposition"

  after_pass_5:
    human_check:
      - "Executive summary coherent"
      - "All risks have owners"
    machine_check:
      - "Traceability matrix > 95% coverage"
      - "Every OPEN risk has recommended action"
    sme_review: "Program director reviews governance report"
```

---

## Chapter 28: Effort Estimation

### Effort Per Strategy

| Strategy | Programs/Week/Analyst | Elapsed (500 programs) | Notes |
|----------|----------------------|----------------------|-------|
| 1: Multi-Pass | 15-25 per pass/week | 4-6 months | 5 passes × 3-4 weeks |
| 2: Single-Pass | 30-50 per week | 2-4 months | Fastest analysis |
| 3: Domain-Driven | 1-2 domains/week | 3-5 months | Depends on domain size |
| 4: Strangler Fig | 1 module/2-4 weeks | 6-18 months | Analysis + dev interleaved |
| 5: Parallel Convert | 40-80 per week | 1-3 months | Heavy review needed |
| 6: Test-First | 20-40 per week (after capture) | 3-6 months | Capture phase is fixed cost |

### API Cost Estimate

At ~$0.05-0.15 per program per pass, a 500-program codebase with 5 passes costs roughly $125-375 in API calls. The cost is trivial compared to human effort.

### LLM Selection

| Factor | Recommendation |
|--------|---------------|
| Context window | 100K+ tokens for large programs |
| COBOL understanding | Test your specific dialect during pilot |
| Structured output | Strong instruction-following for valid YAML |
| Consistency | Temperature 0 or very low |
| Cost optimization | Capable model for complex passes, cheaper for simple ones |

---

## Chapter 29: Risk Checkpoints & Kill Switches

### Checkpoint 1: After Pilot (Week 2)

| Check | Red Flag | Action |
|-------|----------|--------|
| LLM accuracy | Below 70% after tuning | Stop. Different LLM, smaller chunks, or more context. |
| Copybook availability | >20% missing | Stop. Source missing copybooks first. |
| SME availability | Can't get reviews | Escalate. Unvalidated specs are guesses. |
| Effort estimate | Timeline exceeds deadline by 2x+ | Switch to faster strategy or hybrid. |

### Checkpoint 2: After First Domain (Week 4-6)

| Check | Red Flag | Action |
|-------|----------|--------|
| Rule contradictions | >10% contradictory | Normal for old systems. Escalate each to SMEs. |
| Dependency surprises | Far more than expected | Re-assess complexity. May need different strategy. |
| Dev team feedback | Specs insufficient | Add detail. Have developer join analysis team. |
| Spec volume | Nobody reading them | Switch to SDD format. Reduce prose, increase executable specs. |

### Checkpoint 3: After First Java Module (Week 8-12)

| Check | Red Flag | Action |
|-------|----------|--------|
| Parallel run discrepancies | >5% different | Investigate: precision, edge cases, dates. |
| Performance gaps | 10x+ slower | Architecture review. Different patterns needed. |
| Missing functionality | Users report gaps | Add programs to backlog. |
| Scope creep | Improvements mixed with migration | Separate. Modernize first, enhance second. |

### Kill Switches

**Kill Switch 1:** Codebase is unmaintainable even in COBOL — thousands of unreachable paths, contradictory logic. A 1:1 modernization reproduces the mess. Consider full rewrite from requirements.

**Kill Switch 2:** Business processes have changed — COBOL doesn't match how the business actually operates. Switch to requirements-driven development from SME interviews.

**Kill Switch 3:** Analysis cost exceeds 30% of total modernization budget. Simplify approach, reduce scope, or accept higher risk.

---

## Chapter 30: Deliverables Checklist

```
/project-root/
├── docs/                          # Human-readable
│   ├── governance/
│   │   ├── executive-summary.md
│   │   ├── risk-register.md
│   │   └── traceability-report.md
│   ├── business/
│   │   ├── {domain}-process.md
│   │   └── business-rules-catalog.md
│   ├── functional/
│   │   ├── {module}-functional-spec.md
│   │   ├── ui-mapping.md
│   │   └── report-specs.md
│   ├── technical/
│   │   ├── {program}-tech-spec.md
│   │   ├── data-model-design.md
│   │   ├── service-decomposition.md
│   │   ├── error-handling-strategy.md
│   │   └── batch-processing-design.md
│   └── inventory/
│       ├── application-inventory.md
│       ├── dependency-map.md
│       └── data-dictionary.md
│
├── specs/                          # Machine-readable
│   ├── domain-model/
│   │   └── {entity}.schema.yaml
│   ├── api-specs/
│   │   └── {service}.openapi.yaml
│   ├── rules/
│   │   └── {domain}/{ruleset}.rules.yaml
│   ├── features/
│   │   └── {domain}/{feature}.feature
│   ├── workflows/
│   │   └── {process}.workflow.yaml
│   ├── events/
│   │   └── {domain}-events.asyncapi.yaml
│   ├── migration/
│   │   └── {entity}-migration.mapping.yaml
│   └── governance/
│       ├── traceability-matrix.yaml
│       └── risk-register.yaml
│
├── tools/
│   ├── scanner.py
│   ├── assembler.py
│   ├── splitter.py
│   ├── validator.py
│   └── coverage-report.py
│
└── config/
    ├── quality-gates.yaml
    ├── program-tiers.yaml
    └── domain-clusters.yaml
```

### Quick Profile Assessment

Fill this in 15 minutes for a quick strategy recommendation:

```
QUICK PROFILE

Programs:            ___
Estimated LOC:       ___
Batch jobs:          ___
Online transactions: ___
DB2 tables:          ___
CICS maps:           ___
Copybooks:           ___
External interfaces: ___

Industry:    [ ] Banking  [ ] Insurance  [ ] Healthcare
             [ ] Government  [ ] Retail  [ ] Other: ___

Regulation:  [ ] SOX  [ ] Basel  [ ] HIPAA  [ ] PCI  [ ] None

COBOL staff: [ ] In-house  [ ] Contractor  [ ] None
Docs:        [ ] Comprehensive  [ ] Partial  [ ] None
Target:      [ ] Defined  [ ] Partial  [ ] Not defined
Timeline:    [ ] 6 months  [ ] 12 months  [ ] 18+ months
```

---

*End of COBOL Modernization Master Guide*
