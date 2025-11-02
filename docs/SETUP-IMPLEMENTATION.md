# Setup and Implementation Guide - Maven Fuzzy Factory E-Commerce Analytics

## Project Phase: Data Ingestion Complete ✅

**Last Updated:** November 2, 2025
**Status:** Raw data loaded to tables (pass-through dataflows, no transformations)
**Next Phase:** Data quality checks and transformations in Phase 5

---

## 1. Environment Setup Completed

### Microsoft Fabric Workspace
- **Workspace Name:** `Maven-Fuzzy-Factory-Analytics`
- **Lakehouse Name:** `mff_lakehouse`
- **Git Integration:** Enabled for CI/CD (GitHub Actions ready)
- **Capacity:** [Your capacity level]

### GitHub Repository
- **Repository:** `maven-fuzzy-factory-analytics`
- **Visibility:** Public (read-only for viewers)
- **Branch Strategy:** Main branch protected, requires pull request reviews
- **Data Source Attribution:** Maven Analytics - Toy Store E-Commerce Database

---

## 2. Current Data Architecture

### Medallion Architecture (Simplified Implementation)

```
Staging (Raw Files) → Tables (Raw Data as-is) → Gold (Future: Transformations & Aggregations)
```

**Current State:** Data has been loaded as-is from CSV files to Delta Lake tables. No transformations applied yet.

### Current Data Layers

#### **Staging Layer (Files)**
- Location: `mff_lakehouse/Files/staging/`
- Format: CSV files (source backup)
- Purpose: Raw data source and archive
- Files:
  - `website_sessions.csv` (13K+ records)
  - `website_pageviews.csv` (60K+ records)
  - `orders.csv` (3K+ records)
  - `order_items.csv` (4K+ records)
  - `products.csv` (product catalog)
  - `order_item_refunds.csv` (refunds)

#### **Table Layer (Raw Data)**
- Location: `mff_lakehouse/Tables/dbo/`
- Format: Delta Lake tables (optimized for querying)
- Purpose: Raw data stored in queryable format
- Naming Convention: `stg_*` prefix indicates staging/raw layer
- **Status:** Data loaded as-is from CSV files (no transformations)

**Tables Created:**
| Table | Source File | Records | Status |
|-------|------------|---------|--------|
| `stg_sessions` | website_sessions.csv | ~13K | ✅ Loaded as-is |
| `stg_website_pageviews` | website_pageviews.csv | ~60K | ✅ Loaded as-is |
| `stg_orders` | orders.csv | ~3K | ✅ Loaded as-is |
| `stg_order_items` | order_items.csv | ~4K | ✅ Loaded as-is |
| `stg_products` | products.csv | ~Products | ✅ Loaded as-is |
| `stg_order_item_refunds` | order_item_refunds.csv | ~Refunds | ✅ Loaded as-is |
| `ref_data_dictionary` | data-dictionary.csv | Reference | ✅ Loaded as-is |

#### **Gold Layer (Future - Transformations & Business Analytics)**
- Location: TBD (Phase 5)
- Purpose: Cleaned data, aggregations, KPI calculations
- Planned activities:
  - Remove duplicate records
  - Fix data types
  - Handle null values
  - Add derived fields (calculations)
  - Create aggregated tables for Power BI

---

## 3. ETL Pipeline Implementation

### Current Dataflow Architecture
**6 Dataflows Created** (all with Git integration enabled):

| Dataflow | Input | Output | Transformations | Status |
|----------|-------|--------|-----------------|--------|
| `df_sessions_transform` | `website_sessions.csv` | `stg_sessions` | None (pass-through) | ✅ Running |
| `df_pageviews_transform` | `website_pageviews.csv` | `stg_website_pageviews` | None (pass-through) | ✅ Running |
| `df_orders_transform` | `orders.csv` | `stg_orders` | None (pass-through) | ✅ Running |
| `df_order_items_transform` | `order_items.csv` | `stg_order_items` | None (pass-through) | ✅ Running |
| `df_products_transform` | `products.csv` | `stg_products` | None (pass-through) | ✅ Running |
| `df_refunds_transform` | `order_item_refunds.csv` | `stg_order_item_refunds` | None (pass-through) | ✅ Running |

### Current Dataflow Pattern
Each dataflow follows this simple pattern:
1. **Load:** Get data from CSV file in staging folder
2. **Map:** Column mapping (automatic)
3. **Publish:** Write to table in `dbo` schema

```
CSV File → Load Step → Column Auto-Mapping → Delta Table Output
```

### Why Simple Pass-Through Now?
- ✅ **First Priority:** Get data loaded and queryable
- ✅ **Validation:** Can analyze raw data before transformation planning
- ✅ **Staging:** Raw data preserved for analysis and debugging
- ✅ **Flexibility:** Can see data issues before designing transformations

---

## 4. Planned Transformations (Phase 5)

The following transformations are **planned but not yet implemented**:

### Data Quality Issues to Address
- Duplicate records (by primary key)
- Data type inconsistencies (dates stored as strings, etc.)
- Null value handling (how to handle missing data)
- Whitespace and formatting issues
- Invalid values (negative prices, future dates, etc.)

### Planned Transformation Steps (Phase 5)
Each dataflow will be enhanced with:

1. **Deduplication**
   - Remove duplicate records by primary key
   - Example: Remove duplicate `order_id` entries

2. **Data Type Conversion**
   - Ensure correct types: BIGINT (IDs), DATETIME (timestamps), DECIMAL (currency)
   - Fix string dates to proper DateTime format

3. **Null Handling**
   - Text fields: Replace with meaningful defaults ("Direct", "Unknown", etc.)
   - Numeric fields: Replace with 0 or leave as NULL based on business logic
   - Date fields: Handle missing dates appropriately

4. **Column Standardization**
   - Rename to lowercase: `WebsiteSessionID` → `website_session_id`
   - Trim whitespace
   - Remove special characters from text fields

5. **Data Validation**
   - Check for negative values where not allowed
   - Validate date ranges
   - Ensure referential integrity (foreign keys exist)

6. **Derived Fields (Calculated Columns)**
   - Example: `gross_margin = (price_usd - cogs_usd) / price_usd * 100`
   - Create reusable metrics in silver/staging layer

---

## 5. Data Quality Assessment (Current)

### What We Know About the Raw Data
- **Sessions:** 13K+ records, includes UTM parameters, device types, timestamps
- **Pageviews:** 60K+ records, includes page URLs and session IDs
- **Orders:** 3K+ records with pricing and cost information
- **Order Items:** 4K+ records with product associations
- **Products:** Product catalog with launch dates
- **Refunds:** Refund records linked to order items

### Known Observations (From CSV Files)
- [To be populated after data analysis in Phase 5]
- Data types to be confirmed
- Null patterns to be identified
- Duplicate patterns to be assessed

---

## 6. Architecture Decisions Documented

### Decision: Simple Pass-Through Dataflows Initially
- **Rationale:** Load data first, understand structure, plan transformations second
- **Benefit:** Separates data ingestion from data transformation concerns
- **Next Step:** Analyze raw data, then design transformation logic in Phase 5

### Decision: Schema Strategy - `dbo` with Naming Prefix
- **Rationale:** Avoids Fabric Dataflow Gen2 schema discovery limitations
- **Implementation:** Use `stg_*` prefix to indicate staging/raw layer
- **Future:** Can transition to schema-based organization when Fabric improves support

### Decision: Separate Dataflows Per Source
- **Rationale:** One dataflow per CSV file for clarity and modularity
- **Benefit:** Easy to test, modify, and refresh individual sources independently
- **Scalability:** Easy to add new data sources with new dataflows

### Decision: Git Integration Enabled
- **Rationale:** Production-readiness, version control, future CI/CD automation
- **Status:** All dataflows created with Git integration
- **Future:** Ready for GitHub Actions pipeline automation

---

## 7. Current File Organization

### Lakehouse Structure

```
mff_lakehouse/
│
├── 📁 Files/
│   └── 📁 staging/                      # Raw CSV source files
│       ├── website_sessions.csv         # ~13K records
│       ├── website_pageviews.csv        # ~60K records
│       ├── orders.csv                   # ~3K records
│       ├── order_items.csv              # ~4K records
│       ├── products.csv                 # Product catalog
│       ├── order_item_refunds.csv       # Refund data
│       └── maven_fuzzy_factory_data_dictionary.csv  # Reference
│
└── 📁 Tables/
    └── 📁 dbo/                          # All tables in default schema
        ├── stg_sessions                 # Pass-through loaded
        ├── stg_website_pageviews        # Pass-through loaded
        ├── stg_orders                   # Pass-through loaded
        ├── stg_order_items              # Pass-through loaded
        ├── stg_products                 # Pass-through loaded
        ├── stg_order_item_refunds       # Pass-through loaded
        └── ref_data_dictionary          # Pass-through loaded
```

### GitHub Repository Structure

```
maven-fuzzy-factory-analytics/
├── 📁 docs/
│   ├── master-context.md
│   ├── project-overview.md
│   ├── workflow-architecture.md
│   ├── SETUP-IMPLEMENTATION.md          # This document
│   ├── PROJECT-ROADMAP.md
│   └── LESSONS-LEARNED.md
│
├── 📁 data/
│   └── data-dictionary.csv              # Schema reference
│
├── 📁 scripts/                          # Reserved for future Python/SQL scripts
├── 📁 pipelines/                        # Reserved for pipeline configs
├── 📁 dashboards/                       # Reserved for Power BI files
├── 📁 governance/                       # Reserved for governance docs
│
└── 📄 README.md                         # Project overview
```

---

## 8. Next Steps (Phase 5: Data Transformations)

### Phase 5 Objectives
1. **Analyze Raw Data**
   - Identify data quality issues
   - Document null patterns
   - Assess duplicate prevalence
   - Validate data types and ranges

2. **Plan Transformations**
   - Define deduplication logic
   - Decide on data type conversions
   - Design null handling strategy
   - Plan derived fields

3. **Implement Transformations**
   - Add transformation steps to each dataflow
   - Test transformations against requirements
   - Document transformation rationale
   - Monitor data quality metrics

4. **Create Transformation Documentation**
   - Document all transformation logic
   - Create data quality reports
   - Update schema documentation
   - Record edge cases and exceptions

---

## 9. Quick Reference: Table Schemas (As Loaded)

All tables contain columns exactly as they appear in the source CSV files. No column renaming or type conversion has been applied yet.

### `stg_sessions`
Columns: [To be documented after data analysis]
- All columns imported as-is from website_sessions.csv

### `stg_orders`
Columns: [To be documented after data analysis]
- All columns imported as-is from orders.csv

### Other Tables
- `stg_website_pageviews` - As-is from website_pageviews.csv
- `stg_order_items` - As-is from order_items.csv
- `stg_products` - As-is from products.csv
- `stg_order_item_refunds` - As-is from order_item_refunds.csv
- `ref_data_dictionary` - As-is from data-dictionary.csv

**Note:** Schema details will be documented after data type analysis in Phase 5.

---

## 10. Troubleshooting & Common Questions

### Q: Why are the tables created but not transformed?
**A:** Phase 1 prioritizes getting data loaded and accessible. Phase 5 will add the transformation logic. This approach allows us to understand the data first before designing transformations.

### Q: Can I query the tables now?
**A:** Yes! All tables are queryable via:
- SQL Analytics Endpoint (SELECT queries only)
- Power BI direct connection
- Spark notebooks
- Any BI tool connected to Fabric

### Q: When will transformations be applied?
**A:** Phase 5 (planned after Phase 4 completion). This will enhance existing dataflows with:
- Data quality checks
- Deduplication
- Type conversions
- Null handling
- Derived fields

### Q: Will transformations overwrite raw data?
**A:** No. The transformation plan includes:
- Original raw data in `stg_*` tables (staging layer)
- Future transformed data in new tables (gold layer)
- Both versions preserved for audit trail

---

## 11. Resources & Documentation

- **Microsoft Fabric Documentation:** https://learn.microsoft.com/en-us/fabric/
- **Lakehouse Documentation:** https://learn.microsoft.com/en-us/fabric/data-engineering/lakehouse-overview
- **Dataflow Gen2 Guide:** https://learn.microsoft.com/en-us/fabric/data-factory/dataflows-gen2-overview
- **GitHub Repository:** [Your GitHub URL]
- **Project Data Source:** https://mavenanalytics.io/data-playground/toy-store-e-commerce-database

---

## 12. Document History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | Nov 2, 2025 | Initial creation - documented current pass-through state |

---

**Last Updated:** November 2, 2025
**Document Status:** Current for Phase 4 (Data Ingestion)
**Next Update:** After Phase 5 (Data Transformations)
