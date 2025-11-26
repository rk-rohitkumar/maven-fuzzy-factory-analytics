# Phase 6: Transformation Layer - Final Design (HYBRID ARCHITECTURE)

**Project:** Maven Fuzzy Factory E-Commerce Analytics  
**Date:** November 26, 2025 (Updated for Hybrid Architecture)  
**Phase:** 6 (Data Transformation Implementation)  
**Duration:** 10-14 days  

---

## Overview

Phase 6 focuses on transforming staging tables into clean, business-ready intermediate tables **in the Lakehouse**. This phase builds directly on Phase 5's validated data and prepares data for Phase 7's warehouse gold layer.

**Architecture Decision:** Following the **Hybrid Lakehouse + Warehouse** approach:
- **Bronze & Silver (Phase 5-6):** Lakehouse ✅
- **Gold (Phase 7):** Warehouse ✅

---

## Transformation Objectives

| Objective | Approach | Outcome |
|-----------|----------|---------|
| **Deduplication** | Remove duplicate records by primary key | 1 record per natural key |
| **Type Standardization** | Convert data types (strings to dates, decimals, etc.) | Schema-compliant types |
| **Null Handling** | Apply business rules for null values | No null critical fields |
| **Column Standardization** | Normalize naming, casing, whitespace | Consistent naming conventions |
| **Derived Metrics** | Calculate business KPIs | Revenue per session, conversion rates |
| **Referential Integrity** | Validate foreign key relationships | 100% valid relationships |

---

## Hybrid Architecture Data Flow

```
Phase 3-4: Raw Data Loading
LAKEHOUSE: mfflakehouse.dbo.stg_* (Bronze - Raw)
    ↓
Phase 5: Data Quality Validation
LAKEHOUSE: mfflakehouse.dbo.stg_* (Validated 100%)
    ↓
Phase 6: Data Transformation ← WE ARE HERE
LAKEHOUSE: mfflakehouse.dbo.trn_* (Silver - Transformed)
    ↓
Phase 7: Gold Layer Aggregation
WAREHOUSE: mff_warehouse_gold (Gold - BI-ready)
├── analytics.fct_orders
├── analytics.fct_sessions
├── dimensions.dim_products
└── dimensions.dim_date
    ↓
Phase 8: Power BI Consumption
Power BI → Warehouse Endpoint
```

---

## Phase 6 Output Tables (Lakehouse)

### Silver Layer - Transformed Tables

All Phase 6 outputs are **Delta tables in Lakehouse**:

| Input (Bronze) | Output (Silver) | Location | Purpose |
|----------------|-----------------|----------|---------|
| `stg_sessions` | `trn_sessions` | `mfflakehouse.dbo` | Transformed sessions |
| `stg_website_pageviews` | `trn_pageviews` | `mfflakehouse.dbo` | Transformed pageviews |
| `stg_orders` | `trn_orders` | `mfflakehouse.dbo` | Transformed orders |
| `stg_order_items` | `trn_order_items` | `mfflakehouse.dbo` | Transformed order items |
| `stg_products` | `trn_products` | `mfflakehouse.dbo` | Transformed products |
| `stg_order_item_refunds` | `trn_refunds` | `mfflakehouse.dbo` | Transformed refunds |

**Storage:** Lakehouse Delta Tables (optimized for Spark and SQL access)

---

## Transformation Specifications by Table

### 1. stg_sessions → trn_sessions

**Input:** `mfflakehouse.dbo.stg_sessions` (validated in Phase 5)  
**Output:** `mfflakehouse.dbo.trn_sessions` (transformed, ready for aggregation)  
**Storage:** Lakehouse Delta Table

**Key Transformations:**
- Remove duplicate session IDs (keep earliest by created_at)
- Standardize UTM fields (null → 'Direct')
- Convert device_type to uppercase
- Add derived fields:
  - `is_mobile` (boolean: TRUE if device_type = 'mobile')
  - `session_year` (extract year from created_at)
  - `session_month` (extract month from created_at)
  - `session_date` (date portion of created_at)

**Business Rules:**
- Keep only sessions with valid user_id (not null)
- Exclude sessions older than 2010
- Convert repeat_session to explicit boolean (0=FALSE, 1=TRUE)
- Standardize all text fields (trim whitespace, uppercase where appropriate)

**Sample PySpark Code:**
```python
from pyspark.sql.functions import col, when, row_number, upper, coalesce, year, month, to_date
from pyspark.sql.window import Window

# Read staging table from Lakehouse
df = spark.table("mfflakehouse.dbo.stg_sessions")

# Deduplicate by session_id (keep first occurrence)
window_spec = Window.partitionBy("session_id").orderBy("created_at")
df = df.withColumn("rn", row_number().over(window_spec))
df = df.filter(col("rn") == 1).drop("rn")

# Standardize UTM fields (null → 'Direct')
df = df.withColumn("utm_source", coalesce(col("utm_source"), lit("Direct")))
df = df.withColumn("utm_campaign", coalesce(col("utm_campaign"), lit("Direct")))

# Add derived fields
df = df.withColumn("is_mobile", when(col("device_type") == "MOBILE", True).otherwise(False))
df = df.withColumn("session_year", year(col("created_at")))
df = df.withColumn("session_month", month(col("created_at")))
df = df.withColumn("session_date", to_date(col("created_at")))

# Apply business rules
df = df.filter(col("user_id").isNotNull())
df = df.filter(year(col("created_at")) >= 2010)

# Write to Lakehouse transformed table
df.write.mode("overwrite").saveAsTable("mfflakehouse.dbo.trn_sessions")
```

---

### 2. stg_website_pageviews → trn_pageviews

**Input:** `mfflakehouse.dbo.stg_website_pageviews` (validated)  
**Output:** `mfflakehouse.dbo.trn_pageviews` (transformed)  
**Storage:** Lakehouse Delta Table

**Key Transformations:**
- Remove duplicate pageview_id records
- Validate session_id exists in trn_sessions table (referential integrity)
- Add page_sequence (row number within session, ordered by created_at)
- Standardize pageview_url format

**Business Rules:**
- Only include pageviews for valid sessions (exists in trn_sessions)
- Exclude bot traffic (if identifiable via patterns)

---

### 3. stg_orders → trn_orders

**Input:** `mfflakehouse.dbo.stg_orders` (validated)  
**Output:** `mfflakehouse.dbo.trn_orders` (transformed)  
**Storage:** Lakehouse Delta Table

**Key Transformations:**
- Remove duplicate order_id records
- Validate user_id exists in trn_sessions
- Add derived fields:
  - `order_year`, `order_month`, `order_date`
  - `is_first_order` (flag first order per user)

**Business Rules:**
- Only include orders with valid user_id
- Exclude test orders (user_id = 0 or specific test values)

---

### 4. stg_order_items → trn_order_items

**Input:** `mfflakehouse.dbo.stg_order_items` (validated)  
**Output:** `mfflakehouse.dbo.trn_order_items` (transformed)  
**Storage:** Lakehouse Delta Table

**Key Transformations:**
- Remove duplicate order_item_id records
- Validate order_id exists in trn_orders
- Validate product_id exists in trn_products
- Calculate line_total (quantity × unit_price)
- Enrich with product name from trn_products

---

### 5. stg_products → trn_products

**Input:** `mfflakehouse.dbo.stg_products` (validated)  
**Output:** `mfflakehouse.dbo.trn_products` (transformed)  
**Storage:** Lakehouse Delta Table

**Key Transformations:**
- Remove duplicate product_id records
- Standardize product_name (trim, proper case)
- Validate price > 0
- Add is_active flag (default TRUE)

---

### 6. stg_order_item_refunds → trn_refunds

**Input:** `mfflakehouse.dbo.stg_order_item_refunds` (validated)  
**Output:** `mfflakehouse.dbo.trn_refunds` (transformed)  
**Storage:** Lakehouse Delta Table

**Key Transformations:**
- Remove duplicate refund_id records
- Validate order_item_id exists in trn_order_items
- Standardize refund_status (map to standard values)

---

## Implementation Approach

### Option A: Dataflows (Recommended for Simple Transformations)
- Use Power Query in Fabric Dataflows
- Visual transformation interface
- Write to Lakehouse tables
- **Best for:** Deduplication, type conversion, null handling

### Option B: PySpark Notebooks (Recommended for Complex Logic) ✅
- Full control over transformation logic
- Read from/write to Lakehouse
- Reusable code patterns
- **Best for:** Referential integrity, derived metrics, complex calculations

### Option C: Hybrid (Recommended for Production)
- Use Dataflows for simple transformations
- Use Notebooks for complex business logic
- Both write to Lakehouse
- **Best for:** Balanced approach with flexibility

---

## Notebooks for Phase 6

### Recommended Notebook Structure

Create 6 transformation notebooks (one per entity):

| Notebook | Input | Output | Location |
|----------|-------|--------|----------|
| `06_sessions_transform.ipynb` | `stg_sessions` | `trn_sessions` | `notebooks/06-transformation/` |
| `06_pageviews_transform.ipynb` | `stg_website_pageviews` | `trn_pageviews` | `notebooks/06-transformation/` |
| `06_orders_transform.ipynb` | `stg_orders` | `trn_orders` | `notebooks/06-transformation/` |
| `06_orderitems_transform.ipynb` | `stg_order_items` | `trn_order_items` | `notebooks/06-transformation/` |
| `06_products_transform.ipynb` | `stg_products` | `trn_products` | `notebooks/06-transformation/` |
| `06_refunds_transform.ipynb` | `stg_order_item_refunds` | `trn_refunds` | `notebooks/06-transformation/` |

**All notebooks:**
- Read from `mfflakehouse.dbo.stg_*`
- Write to `mfflakehouse.dbo.trn_*`
- Attach to Lakehouse workspace

---

## Phase 7 Preparation (Warehouse)

### What Happens After Phase 6?

Once transformation completes, **Phase 7 creates the Warehouse gold layer**:

```
Lakehouse Silver (trn_*)
    ↓ ETL Pipeline
Warehouse Gold (fct_*, dim_*)
```

**Phase 7 Activities:**
1. Create Warehouse: `mff_warehouse_gold`
2. Design schemas: `analytics`, `dimensions`
3. Build ETL pipelines: Lakehouse → Warehouse
4. Create fact tables (`fct_orders`, `fct_sessions`)
5. Create dimension tables (`dim_products`, `dim_date`)
6. Test Power BI connection to warehouse

**Phase 6 prepares the data for Phase 7 aggregation.**

---

## Success Criteria for Phase 6

When Phase 6 is complete:

- [ ] All 6 `trn_*` tables created in Lakehouse
- [ ] 0 duplicates in output tables
- [ ] 0 null values in critical fields
- [ ] All foreign key relationships valid
- [ ] Transformation logic documented
- [ ] Derived metrics calculated correctly
- [ ] Tables ready for Phase 7 warehouse aggregation
- [ ] Pipeline execution time < 15 minutes

---

## Architecture Comparison

### What's Different with Hybrid Architecture?

| Aspect | Original Plan (Lakehouse Only) | New Plan (Hybrid) |
|--------|-------------------------------|-------------------|
| **Phase 6 Output** | `trn_*` in Lakehouse | ✅ Same - `trn_*` in Lakehouse |
| **Phase 7 Output** | `fct_*`, `dim_*` in Lakehouse | ✅ **`fct_*`, `dim_*` in Warehouse** |
| **Power BI Source** | Lakehouse SQL endpoint | ✅ **Warehouse endpoint** |
| **Performance** | Good | ✅ **Excellent (20-50% faster)** |
| **Security** | Good | ✅ **Better (RLS, CLS built-in)** |
| **Cost** | Lower | ✅ **~5-10% increase** |

**Phase 6 itself doesn't change** - only Phase 7 destination changes.

---

## Next Steps

1. **Choose transformation approach** (A, B, or C) - Recommend Option B (Notebooks)
2. **Create transformation notebooks** (2-3 days)
   - One notebook per entity
   - Follow naming convention: `06_<entity>_transform.ipynb`
   - All write to Lakehouse `trn_*` tables
3. **Test with sample data** (1 day)
4. **Integrate into pipeline** (1 day)
   - Add transformation activities after validation
   - Configure to run only if validation PASSED
5. **Validate full pipeline** (1 day)
6. **Document and review** (1 day)

---

**Owner:** Data Engineering Team  
**Created:** November 25, 2025  
**Updated:** November 26, 2025 (Hybrid Architecture)  
**References:** 
- `Data_Naming_Conventions_HYBRID.md`
- `Lakehouse_vs_Warehouse_Architecture_Decision.md`
