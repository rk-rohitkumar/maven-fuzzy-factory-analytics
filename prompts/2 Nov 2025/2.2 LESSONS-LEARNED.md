# Lessons Learned & Technical Insights - Maven Fuzzy Factory Project

**Document Date:** November 2, 2025
**Audience:** Data Engineers, Analytics Engineers, Team Members

---

## Executive Summary

This document captures real-world learnings from implementing the Maven Fuzzy Factory E-Commerce Analytics project in Microsoft Fabric. It documents practical decisions, technical constraints encountered, and pragmatic solutions adopted. 

**Current Phase:** Data ingestion complete. Raw data loaded as-is to tables. No transformations applied yet.

---

## 1. Microsoft Fabric Setup & Configuration

### Lesson 1.1: Always Reference Official Documentation
**Experience:** Initial instructions assumed capabilities that don't exist in current Fabric UI.

**Key Learning:**
- Always reference official Microsoft Learn documentation when making Fabric environment changes
- Community discussions (Reddit, LinkedIn) often identify known limitations
- Features may be in Preview, GA, or Deprecated - check current status before implementing

**Action Taken:**
- Established practice of verifying features against: https://learn.microsoft.com/fabric/
- Document official sources for all Fabric configuration decisions
- Report any inaccuracies discovered during implementation

---

### Lesson 1.2: Workspace Organization Strategy
**Experience:** Created workspace successfully; schema organization challenges appeared during dataflow phase.

**Best Practice:**
- Workspace name should reflect project context: `Maven-Fuzzy-Factory-Analytics`
- Use clear, meaningful naming conventions across all resources
- Create workspace-level documentation explaining structure
- Enable Git integration at workspace level from the start

**Naming Convention Established:**
```
Workspaces:     [Project-Name-Analytics]
Lakehouses:     [project_lakehouse]
Dataflows:      df_[source]_[action]
Tables:         [layer]_[source]
Columns:        lowercase_with_underscores
```

---

## 2. Data Architecture Decisions

### Lesson 2.1: Schema Organization - Pragmatism Over Perfection
**Challenge:** Attempted to implement textbook medallion architecture with custom schemas.

**Issue Discovered:**
- Dataflow Gen2 **cannot discover or publish to custom schemas** (confirmed limitation)
- Only `dbo` schema appears in Dataflow destination selector
- Tables in custom schemas are invisible to Dataflow UI

**Decision Made:**
- Use `dbo` schema with **naming prefixes** to indicate data layer
- `stg_*` = Staging layer (raw data, loaded as-is)
- `gold_*` = Gold layer (future: transformed and aggregated)
- `ref_*` = Reference/dimension tables

**Impact:**
- ✅ Cleaner implementation
- ✅ Fewer manual steps
- ✅ Better maintainability
- ✅ Aligns with enterprise practice
- ⚠️ Dependent on naming discipline
- ⚠️ Less elegant than schema-based organization

**Real-World Insight:**
Many production data warehouses operate this way. Schema separation is nice-to-have, not must-have.

---

### Lesson 2.2: Files vs. Tables - Clear Separation
**Learning:** Microsoft Fabric distinguishes storage concepts clearly:

| Aspect | Files | Tables |
|--------|-------|--------|
| **Purpose** | Raw archive, backup | Query-optimized data |
| **Format** | CSV, Parquet, JSON | Delta Lake |
| **Storage Location** | `/Files` folder | `/Tables` schema |
| **Query Performance** | Slow (scans entire file) | Fast (indexed, optimized) |
| **Power BI Connection** | Requires special setup | Direct connection |

**Decision:** Keep staging files for archival backup, load all data to tables for analytics.

---

### Lesson 2.3: Bronze Layer Necessity Assessment
**Question:** Why have bronze layer if we skip to staging?

**Answer:** Bronze layer is **optional overhead** for your current use case.

**Use Bronze When:**
- ✅ Multiple source systems (need to audit raw data from each)
- ✅ Real-time streaming (need replay capability)
- ✅ Strict compliance requirements (preserve exact raw state)
- ❌ Single CSV source (files already serve as backup)

**Decision for This Project:** Skip bronze layer.
- Staging CSV files serve as raw data backup
- Reduces architecture complexity
- Can add later if multi-source requirements emerge

---

### Lesson 2.4: Transformation Phasing Strategy
**Insight:** Separate data loading from data transformation.

**Why Phase 1: Load as-is**
- ✅ Get data queryable immediately
- ✅ Analyze raw data structure before designing transformations
- ✅ Identify data quality issues
- ✅ Plan transformation logic based on actual data

**Why Phase 5: Add Transformations**
- ✅ After understanding data characteristics
- ✅ Based on observed issues, not assumptions
- ✅ Can test transformations against real patterns
- ✅ Documentation reflects actual transformation needs

**Benefit:** Separates concerns and reduces rework.

---

## 3. ETL Pipeline Implementation

### Lesson 3.1: Dataflow Gen2 with Git Integration
**Configuration:** Enable Git integration, deployment pipelines, and Public API scenarios.

**Why Enable:**
- ✅ Version control for dataflow definitions
- ✅ Ready for CI/CD pipelines
- ✅ Can trigger refreshes via API
- ✅ Production-ready approach from day one
- ⏱️ Adds ~30 seconds to creation (minimal overhead)

**Recommendation:** Always enable unless building throwaway prototypes.

---

### Lesson 3.2: Pass-Through Dataflows as Staging
**Design Pattern:** Simple load-as-is pattern for initial data ingestion.

**Pattern:**
```
CSV File → Load Step → Column Auto-Mapping → Delta Table
```

**Benefits:**
- ✅ Fast to implement (no transformation logic required)
- ✅ Preserves original data exactly
- ✅ Easy to debug (what you see is what you get)
- ✅ Baseline for transformation planning

**When to Use:**
- ✅ Data discovery phase (need to see raw data)
- ✅ Staging layer (preserve original as-is)
- ✅ Rapid prototyping (get data flowing first)

**When to Add Transformations:**
- After data quality assessment
- After business requirements clarified
- After schema and nulls analyzed

---

### Lesson 3.3: Dataflow Refresh Configuration
**Experience:** Dataflows running successfully on schedule.

**Setup Pattern:**
1. Create dataflow (with Git integration enabled)
2. Test single run (verify data loads)
3. Configure refresh schedule (nightly, hourly, etc.)
4. Monitor success/failure in history

**Monitoring Indicators:**
- ✅ All 6 dataflows executing successfully
- ✅ Record counts stable (no unexpected changes)
- ✅ No error messages in dataflow logs
- ✅ Refresh times consistent

---

## 4. Current Project State

### What's Been Done ✅

**Data Ingestion:**
- 6 CSV files uploaded to staging folder (~80K total records)
- 7 tables created in `dbo` schema with `stg_` prefix
- All dataflows running and refreshing successfully
- Data readable and queryable

**Current Architecture:**
```
Staging (CSV Files) → Dataflows (Pass-through) → Tables (Raw Data)
```

### What's NOT Been Done (Yet) ⏳

**Transformations (Planned Phase 5):**
- ❌ Deduplication (by primary key)
- ❌ Data type conversion
- ❌ Null value handling
- ❌ Column standardization (lowercase naming)
- ❌ Derived field calculations
- ❌ Data validation and cleansing

### Why This Approach?

**Rationale:**
1. **Learn First, Transform Second** - Analyze raw data before designing transformations
2. **Identify Issues** - See what data quality problems actually exist
3. **Document Decisions** - Base transformation logic on observed patterns, not assumptions
4. **Reduce Rework** - Changes to transformation logic don't require reloading raw data

---

## 5. Common Issues Encountered & Solutions

### Issue 5.1: "CREATE TABLE Not Supported" in SQL Analytics Endpoint
**Symptom:** Tried to create tables via SQL, got error.

**Root Cause:** SQL Analytics Endpoint is **read-only** - designed for queries, not DDL.

**Solution:**
- Use Apache Spark in Notebooks for table creation
- Use Dataflows for table creation from transformations
- SQL Endpoint only for SELECT queries

**Learning:** Different tools have different capabilities. Understand purpose of each tool.

---

### Issue 5.2: Dataflow Cannot Find Tables in Custom Schema
**Symptom:** Tables created in custom schema don't appear in Dataflow destination picker.

**Root Cause:** Dataflow Gen2 limitation - only discovers `dbo` schema.

**Workaround (Used):** Use `dbo` schema with naming prefix (`stg_*`)

**Alternatives (Not Used):**
- Search for "existing table" when table exists
- Create tables in Python, then move via Notebook (manual)
- Wait for Microsoft improvement (timeline unknown)

**Lesson:** When hitting limitations, find pragmatic workarounds. Don't fight the system.

---

### Issue 5.3: Schema vs. Naming Prefixes Debate
**Challenge:** How to organize data layers?

**Options Considered:**
1. Custom schemas (`silver`, `gold`)
2. Naming prefixes (`stg_*`, `gold_*`)
3. Separate lakehouses per layer

**Chosen:** Naming prefixes in `dbo` schema

**Rationale:** Works with current Fabric capabilities, widely used in enterprise.

**Trade-off:** Less elegant but more practical.

---

## 6. Best Practices Established

### 6.1 Naming Conventions
```
Workspaces:     [Project-Name]-[Type]            Maven-Fuzzy-Factory-Analytics
Lakehouses:     [project]_lakehouse              mff_lakehouse
Dataflows:      df_[source]_[action]             df_sessions_transform
Tables:         [layer]_[source]                 stg_sessions, gold_metrics
Columns:        lowercase_with_underscores       website_session_id
```

### 6.2 Documentation Requirements
- Every dataflow needs transformation logic documented
- Every table needs schema documentation
- Every decision needs recorded rationale
- Share learnings via GitHub and LinkedIn

### 6.3 Quality Standards (To Implement Phase 5)
- [ ] No unexpected duplicates (assess after loading)
- [ ] Correct data types (validate in Phase 5)
- [ ] Null values handled appropriately (design Phase 5)
- [ ] Traceable data lineage (document Phase 5)

### 6.4 Git Integration
- ✅ All dataflows created with Git integration
- ✅ Version control enabled
- ✅ Ready for CI/CD GitHub Actions
- ⏳ CI/CD pipelines to implement Phase 7

---

## 7. What Worked Well ✅

### ✅ Staged Approach (Load First, Transform Later)
Benefits realized:
- Got data flowing quickly
- Can analyze before designing transformations
- Reduced assumptions about data quality

### ✅ Simple Pass-Through Dataflows
Benefits realized:
- Fast to implement
- Easy to debug
- Maintains original data integrity

### ✅ Pragmatic Architecture Decisions
Benefits realized:
- Using naming conventions instead of fighting schema limitations
- Simplified complexity while maintaining functionality
- Reduced time to value

### ✅ Git Integration Enabled
Benefits realized:
- Version control built-in
- Ready for automation
- Enterprise-ready from start

### ✅ Clear Documentation
Benefits realized:
- Easy to onboard new team members
- Decisions recorded for future reference
- Community learnings captured

---

## 8. What to Improve (Next Time)

### 📝 Transformation Planning
- **Could be Better:** Design transformation requirements upfront (partially done)
- **Next Time:** Create detailed transformation specification document before coding

### 🔍 Data Quality Assessment
- **Could be Better:** Analyze data earlier in process
- **Next Time:** Add data profiling notebook immediately after loading

### 📊 Stakeholder Communication
- **Could be Better:** Update stakeholders more frequently
- **Next Time:** Weekly progress reports with data samples

### 📚 Documentation Completeness
- **Could be Better:** Document as you go, not at end
- **Next Time:** Live documentation updated during implementation

---

## 9. Planned Transformation Work (Phase 5)

### Data Analysis Tasks
1. Analyze each table for:
   - Duplicate records (by primary key)
   - Null value patterns
   - Data type mismatches
   - Invalid values (negative prices, future dates, etc.)
   - Whitespace and formatting issues

2. Document findings:
   - Issues per table
   - Severity assessment
   - Transformation requirements
   - Edge cases and exceptions

### Transformation Implementation
1. Enhance each dataflow with:
   - Deduplication logic
   - Type conversions
   - Null handling (business-logic based)
   - Column standardization
   - Validation rules

2. Create derived fields:
   - Example: `gross_margin` in orders
   - Example: `conversion_flag` in sessions
   - Example: `revenue_per_session` calculations

3. Test transformations:
   - Verify record counts
   - Check data types
   - Validate business rules
   - Document test results

---

## 10. Key Insights for Community Sharing

### 1. Separate Loading from Transformation
- Get data flowing first
- Analyze before transforming
- Reduces assumptions and rework

### 2. Pragmatism Over Architecture Purity
- Name prefixes work as well as schemas
- Naming conventions are powerful tools
- Real-world constraints matter

### 3. Git Integration for Data Pipelines
- Changes everything for collaboration
- Enables automation
- Should be default practice

### 4. Staged Approach to Analytics
- Phase 1: Load as-is
- Phase 2-3: Understand and plan
- Phase 4-5: Transform and refine

---

## 11. Recommendations for Future Phases

### Phase 5 (Data Transformation)
- Analyze raw data quality issues first
- Document transformation requirements
- Use same naming convention: `stg_*`, `gold_*`
- Create test suite for transformations

### Phase 6 (Power BI Dashboards)
- Connect to gold layer (transformed data)
- Implement row-level security if needed
- Performance optimization (aggregations)

### Phase 7 (Automation)
- GitHub Actions for dataflow refresh
- Notifications for failures
- Data quality alerts

### Phase 8 (Documentation & Sharing)
- Blog post: "Pragmatic Data Architecture in Fabric"
- LinkedIn article: "Real-world learnings from analytics project"
- GitHub Wiki: Complete setup guide

---

## 12. Resources & References

### Official Microsoft Documentation
- Fabric: https://learn.microsoft.com/fabric/
- Lakehouse: https://learn.microsoft.com/fabric/data-engineering/lakehouse-overview
- Dataflow Gen2: https://learn.microsoft.com/fabric/data-factory/dataflows-gen2-overview
- Schemas: https://learn.microsoft.com/fabric/data-engineering/lakehouse-schemas

### Project Resources
- Data Source: https://mavenanalytics.io/data-playground/toy-store-e-commerce-database
- GitHub: [Your project repository]

---

**Document Version:** 1.1
**Last Updated:** November 2, 2025 (Updated for pass-through dataflow phase)
**Status:** Complete for Phase 4
