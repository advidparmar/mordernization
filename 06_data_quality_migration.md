# Pass 6 — Data Quality & Migration Readiness

## Purpose

Go beyond data *structure* analysis (done in Pass 1/4) to analyze actual data *quality*, production data patterns, and migration edge cases. This pass produces the detailed migration plan with validation scripts, not just field mappings.

## Why This Pass Exists

The #1 cause of migration failures is bad data. Every legacy COBOL system has:
- Fields that should never be null but are (because the validation was added after the data was created)
- Date fields with sentinel values (00000000, 99999999, 00010101) that mean different things
- Numeric fields containing spaces (COBOL allows this, Java doesn't)
- Records that violate the business rules extracted in Pass 2
- Orphaned records with no parent
- Duplicate records that the COBOL system handles via "first match wins"
- Character encoding issues (EBCDIC special characters, packed decimal zones)

If your coding agent has access to sample data files or database extracts in the repo, this pass can profile them. If not, it generates the profiling scripts and validation queries that your DBA should run.

## Inputs Required

- Domain model schemas (Pass 1)
- Migration mappings (Pass 4)
- Business rules (Pass 2) — to identify which rules data might violate
- Sample data files (if available in repo)
- DB2 DDL (if available)
- Data classification (Pass 5)

## Outputs Produced

| Output | Human-Readable | Machine-Readable |
|--------|---------------|-----------------|
| Data Quality Report | `06_data_quality/prose/DATA_QUALITY_REPORT.md` | `06_data_quality/specs/data_quality_profile.yaml` |
| Migration Plan | `06_data_quality/prose/MIGRATION_PLAN.md` | `06_data_quality/specs/migration_plan.yaml` |
| Validation Scripts | — | `06_data_quality/scripts/` |

---

## Prompt 6.1 — Data Quality Analysis & Profiling Scripts

```
ROLE: You are a data migration specialist who has seen dozens of COBOL 
migrations fail due to data quality issues. Your job is to anticipate 
every way the data can be problematic and create validation scripts.

CONTEXT:
- System: {{SYSTEM_NAME}}
- Domain: {{DOMAIN_CONTEXT}}

PRIOR CONTEXT:
- Domain models: {{OUTPUT_DIR}}/01_discovery/specs/domain-model/
- Migration mappings: {{OUTPUT_DIR}}/04_technical/specs/migration/
- Business rules: {{OUTPUT_DIR}}/02_business/specs/rules/
- Data classification: {{OUTPUT_DIR}}/05_security/specs/data_classification.yaml

TASK: For each entity in the domain model, analyze the COBOL data 
definitions and business rules to identify every possible data quality 
issue. Then generate SQL scripts (for DB2) or file-processing scripts 
that should be run against production data BEFORE migration.

===== PRODUCE THREE OUTPUTS =====

FILE 1: {{OUTPUT_DIR}}/06_data_quality/prose/DATA_QUALITY_REPORT.md

## Data Quality Assessment — {{SYSTEM_NAME}}

### Executive Summary
[2-3 paragraphs: Overall data quality risk assessment. How confident 
are we that migration will succeed without data issues?]

### Entity-by-Entity Risk Analysis

For each entity:

**[Entity Name]** — Risk: [LOW / MEDIUM / HIGH / VERY HIGH]

| Field | Risk | Issue | Impact | Remediation |
|---|---|---|---|---|
| [field] | [H/M/L] | [e.g., "PIC 9(8) date field may contain 00000000, 99999999, or invalid dates like 20241340"] | [What breaks in Java if not handled] | [How to fix — map to null, reject, default] |

Common patterns to check for EVERY entity:
- **Null/Space Analysis:** Which fields contain spaces where the Java 
  model expects non-null? What percentage of records?
- **Date Validity:** Are all date fields valid dates? What sentinel 
  values exist? Are there impossible dates (month 13, day 32)?
- **Numeric Validity:** Do any numeric fields contain non-numeric data? 
  (COBOL allows moving spaces to numeric fields)
- **Referential Integrity:** Do all foreign key references resolve? 
  Are there orphaned child records?
- **Business Rule Compliance:** How many existing records violate the 
  business rules from Pass 2? (These records exist because the rules 
  were added or changed after the data was created)
- **Duplicate Detection:** Are there duplicate records by business key? 
  How does the COBOL system handle duplicates?
- **Encoding Issues:** Any fields with EBCDIC-specific characters that 
  won't map cleanly to UTF-8?
- **Packed Decimal Integrity:** Any COMP-3 fields with invalid zone 
  nibbles? (Corrupted packed decimal data crashes Java converters)
- **Maximum Value Analysis:** Any fields approaching their PIC clause 
  maximum? (If balance is PIC S9(7)V99 and some accounts are at 
  $9,999,999.99, you have overflow risk)

### Data Volume Analysis
| Entity | Estimated Rows | Record Size | Total Size | Migration Time Estimate |
|---|---|---|---|---|
| [entity] | [N] | [bytes] | [GB] | [hours at estimated throughput] |

### Migration Sequence Recommendation
[Order entities should be migrated, considering referential integrity]
1. [Reference/lookup tables first]
2. [Parent entities next]
3. [Child entities last]
4. [Transaction/history tables — may need special handling]

### Data Archival Recommendation
[Which data can be archived vs. migrated? Old historical records, 
closed accounts beyond retention period, etc.]

---

FILE 2: {{OUTPUT_DIR}}/06_data_quality/specs/data_quality_profile.yaml

```yaml
data_quality:
  system: "{{SYSTEM_NAME}}"
  
  entities:
    - entity: "[entity name]"
      schema_ref: "[domain-model schema path]"
      migration_ref: "[migration mapping path]"
      estimated_rows: [N]
      overall_risk: "[LOW | MEDIUM | HIGH | VERY_HIGH]"
      
      field_risks:
        - field: "[field name]"
          risk: "[HIGH | MEDIUM | LOW]"
          issues:
            - type: "[NULL_IN_REQUIRED | INVALID_DATE | SPACES_IN_NUMERIC | REFERENTIAL_ORPHAN | RULE_VIOLATION | DUPLICATE | ENCODING | OVERFLOW | PACKED_DECIMAL_CORRUPT]"
              description: "[specific issue]"
              estimated_frequency: "[percentage or count if known]"
              impact: "[what breaks]"
              remediation:
                strategy: "[MAP_TO_NULL | REJECT | DEFAULT_VALUE | TRANSFORM | MANUAL_REVIEW]"
                detail: "[specific fix]"
          
          profiling_checks:
            - check: "null_count"
              query: "[SQL or script to count nulls/spaces]"
            - check: "distinct_values"
              query: "[SQL to get value distribution]"
            - check: "min_max"
              query: "[SQL for range analysis]"
            - check: "pattern_match"
              query: "[SQL to find invalid patterns]"

      referential_checks:
        - parent_entity: "[entity]"
          parent_field: "[field]"
          child_field: "[field]"
          check_query: "[SQL for orphan detection]"

      duplicate_checks:
        - business_key: ["field1", "field2"]
          check_query: "[SQL for duplicate detection]"
          expected_result: "[should be zero / known duplicates exist]"

  migration_sequence:
    - order: 1
      entities: ["reference tables"]
      rationale: "[why first]"
    - order: 2
      entities: ["parent entities"]
      rationale: "[referential integrity]"

  archival_candidates:
    - entity: "[entity]"
      criteria: "[e.g., records older than 7 years]"
      estimated_rows: [N]
      rationale: "[why archive instead of migrate]"
```

---

FILE 3: {{OUTPUT_DIR}}/06_data_quality/scripts/

Generate actual runnable SQL scripts:

```sql
-- File: validate_[entity].sql
-- Run against production DB2 BEFORE migration

-- 1. Row count baseline
SELECT COUNT(*) AS total_rows FROM [table];

-- 2. Null/space analysis per field
SELECT 
  COUNT(*) AS total,
  SUM(CASE WHEN [field] IS NULL OR TRIM([field]) = '' THEN 1 ELSE 0 END) AS null_or_empty,
  DECIMAL(SUM(CASE WHEN [field] IS NULL OR TRIM([field]) = '' THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 5, 2) AS pct
FROM [table];

-- 3. Date validation
SELECT COUNT(*) AS invalid_dates
FROM [table]
WHERE [date_field] NOT IN (0, 99999999)
  AND (
    SUBSTR(CHAR([date_field]), 5, 2) NOT BETWEEN '01' AND '12'
    OR SUBSTR(CHAR([date_field]), 7, 2) NOT BETWEEN '01' AND '31'
  );

-- 4. Business rule violation check
-- BR-V-001: Account status must be A, D, S, C, or X
SELECT [status_field], COUNT(*) 
FROM [table]
WHERE [status_field] NOT IN ('A', 'D', 'S', 'C', 'X')
GROUP BY [status_field];

-- 5. Financial totals (for post-migration reconciliation)
SELECT 
  SUM([balance_field]) AS total_balance,
  COUNT(*) AS record_count
FROM [table];

-- 6. Referential integrity
SELECT COUNT(*) AS orphaned_records
FROM [child_table] c
LEFT JOIN [parent_table] p ON c.[fk_field] = p.[pk_field]
WHERE p.[pk_field] IS NULL;

-- 7. Duplicate detection
SELECT [business_key_fields], COUNT(*) AS dup_count
FROM [table]
GROUP BY [business_key_fields]
HAVING COUNT(*) > 1;
```
```

---

## Prompt 6.2 — Comprehensive Migration Plan

```
ROLE: You are a data migration architect producing the definitive 
migration plan that the migration team will execute.

TASK: Using all data quality analysis and migration mappings, produce 
the complete migration plan.

Read:
- All migration mappings: {{OUTPUT_DIR}}/04_technical/specs/migration/
- Data quality profile: {{OUTPUT_DIR}}/06_data_quality/specs/data_quality_profile.yaml
- Data classification: {{OUTPUT_DIR}}/05_security/specs/data_classification.yaml

===== PRODUCE TWO FILES =====

FILE 1: {{OUTPUT_DIR}}/06_data_quality/prose/MIGRATION_PLAN.md

## Data Migration Plan — {{SYSTEM_NAME}}

### Migration Strategy
[Big bang vs. phased? Downtime window? Dual-write period?]

### Pre-Migration Checklist
1. [ ] Run all validation scripts from `06_data_quality/scripts/`
2. [ ] Review and resolve data quality issues
3. [ ] Baseline all financial totals for reconciliation
4. [ ] Archive data beyond retention period
5. [ ] Verify target schema is deployed
6. [ ] Test migration with sample data (1% extract)

### Migration Sequence
[Ordered list of entities with dependencies, estimated time, and 
rollback approach for each]

### Post-Migration Validation
[Step-by-step validation checklist — row counts, financial totals, 
sample comparison, referential integrity]

### Rollback Plan
[How to roll back if migration fails at each stage]

---

FILE 2: {{OUTPUT_DIR}}/06_data_quality/specs/migration_plan.yaml
[Machine-readable version of the plan]
```

---

## Quality Gate Checklist

- [ ] Every entity has a data quality risk assessment
- [ ] Validation SQL scripts are generated and syntactically valid
- [ ] Migration sequence respects referential integrity
- [ ] Financial reconciliation checks are defined with zero tolerance
- [ ] Archival candidates are identified
- [ ] Sensitive data handling during migration is addressed (from data classification)
- [ ] Rollback plan exists for every migration stage
