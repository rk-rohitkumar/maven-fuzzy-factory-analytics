# Data Naming Conventions & Architecture Guidelines

**Project:** Maven Fuzzy Factory E-Commerce Analytics  
**Date:** November 26, 2025  
**Purpose:** Establish consistent naming conventions across all data layers and artifacts

---

## Overview

This document defines standardized naming conventions for all tables, objects, and artifacts in the Maven Fuzzy Factory analytics project. Consistent naming improves discoverability, maintainability, and clarity across the team.

---

## Table Naming Convention

### Medallion Architecture Layers

The project follows a **Medallion Architecture** with clearly defined layers:

```
CSV Files (Raw Source)
    ↓
stg_* (Staging - Raw Data as-is)
    ↓
val_* (Validated - After quality checks)
    ↓
trn_* (Transformed - Business logic applied)
    ↓
gold_* (Gold Layer - Aggregated & Curated)
    ├── fct_* (Fact Tables)
    ├── dim_* (Dimension Tables)
    └── mart_* (Data Marts - Domain-specific)
```

### Table Naming Format

**Format:** `<prefix>_<domain>_<entity>`

**Pattern Breakdown:**
- `<prefix>` - Layer indicator (stg, val, trn, gold, fct, dim, mart)
- `<domain>` - Business domain or system (optional for small tables)
- `<entity>` - Core entity name (singular)

### Layer-Specific Naming Rules

#### **Stage Layer (`stg_*`)**
**Purpose:** Raw data loaded as-is from source systems  
**Location:** Lakehouse - `mfflakehouse.dbo`  
**Naming:** `stg_<entity>`

Examples:
- `stg_sessions` - Raw website session data
- `stg_orders` - Raw order transactions
- `stg_products` - Raw product catalog
- `stg_website_pageviews` - Raw page view logs
- `stg_order_items` - Raw order line items
- `stg_order_item_refunds` - Raw refund records

**Rules:**
- Use snake_case (lowercase with underscores)
- No version suffixes (v1, v2)
- Match source table names when possible
- Keep names concise but descriptive

#### **Validated Layer (`val_*`)**
**Purpose:** Data after quality validation passes (Phase 5 output)  
**Location:** Lakehouse - `mfflakehouse.dbo`  
**Naming:** `val_<entity>`
**Status:** Post-validation, pre-transformation

Examples:
- `val_sessions` - Sessions after quality checks
- `val_orders` - Orders validated
- `val_products` - Products validated

**Rules:**
- Prefix indicates validation has passed
- Same entity name as source staging table
- Used as source for Phase 6 transformations

#### **Transform Layer (`trn_*`)**
**Purpose:** Data after business logic transformation (Phase 6 output)  
**Location:** Lakehouse - `mfflakehouse.dbo`  
**Naming:** `trn_<entity>`
**Status:** Cleaned, deduplicated, type-standardized

Examples:
- `trn_sessions` - Sessions transformed (deduplicated, types fixed, nulls handled)
- `trn_orders` - Orders transformed
- `trn_order_items` - Order items transformed
- `trn_products` - Products transformed
- `trn_website_pageviews` - Pageviews transformed
- `trn_order_item_refunds` - Refunds transformed

**Rules:**
- Indicates transformations applied (dedup, type conversion, null handling)
- Same entity name as staging table (no "_transformed" suffix)
- Ready for aggregation in Phase 7

#### **Gold Layer (`gold_*`, `fct_*`, `dim_*`)**
**Purpose:** Business-ready data for reporting and analytics (Phase 7)  
**Location:** Lakehouse - `mfflakehouse.dbo`  
**Naming:** `gold_<purpose>` or `fct_<entity>` or `dim_<entity>`

Examples (Fact Tables):
- `fct_orders` - Order facts with metrics
- `fct_sessions` - Session conversion facts
- `fct_daily_sales` - Aggregated daily sales

Examples (Dimension Tables):
- `dim_products` - Product dimension
- `dim_date` - Time dimension
- `dim_customer` - Customer dimension
- `dim_marketing_channel` - Marketing channel dimension

Examples (Data Marts):
- `mart_sales_analytics` - Sales domain mart
- `mart_marketing_performance` - Marketing domain mart
- `mart_customer_360` - Customer domain mart

**Rules:**
- Use clear, business-friendly names
- Fact tables use `fct_` prefix
- Dimensions use `dim_` prefix
- Data marts use `mart_` prefix
- Include business context in name

---

## Column Naming Convention

### Column Naming Format

**Format:** `<entity>_<attribute>` or `<attribute>` (depending on context)

**General Rules:**
- Use snake_case (lowercase with underscores)
- Be descriptive but concise
- Avoid abbreviations unless universally understood
- Use consistent terminology across tables

### Column Naming Examples

| Table | Column | Pattern | Notes |
|-------|--------|---------|-------|
| `stg_sessions` | `session_id` | `<entity>_id` | Primary key |
| `stg_sessions` | `user_id` | `<entity>_id` | Foreign key |
| `stg_sessions` | `created_at` | `<attribute>_at` | Timestamp |
| `stg_orders` | `order_date` | `<entity>_date` | Date field |
| `stg_products` | `product_name` | `<entity>_name` | Text field |
| `stg_order_items` | `unit_price` | `unit_<attribute>` | Currency |
| `trn_sessions` | `is_mobile` | `is_<boolean>` | Boolean flag |
| `trn_sessions` | `session_year` | `<entity>_<period>` | Derived date |

### Standard Column Suffixes

| Suffix | Data Type | Example | Meaning |
|--------|-----------|---------|---------|
| `_id` | INT/BIGINT | `session_id` | Primary/foreign key |
| `_at` | TIMESTAMP | `created_at` | Timestamp |
| `_date` | DATE | `order_date` | Date value |
| `_count` | INT | `page_count` | Count/quantity |
| `_amount` | DECIMAL | `order_amount` | Currency amount |
| `_rate` | DECIMAL | `conversion_rate` | Percentage/rate |
| `_flag` | BOOLEAN | `is_completed` | Boolean status |
| `_code` | STRING | `product_code` | Code/identifier |
| `_text` | STRING | `description` | Text content |

---

## Schema Organization

### Lakehouse Schema Strategy

All tables are organized in the default `dbo` schema:
- `mfflakehouse.dbo.stg_*`
- `mfflakehouse.dbo.val_*`
- `mfflakehouse.dbo.trn_*`
- `mfflakehouse.dbo.gold_*`
- `mfflakehouse.dbo.fct_*`
- `mfflakehouse.dbo.dim_*`

**Rationale:**
- Single schema reduces complexity
- Layer prefix (`stg_`, `trn_`, etc.) provides logical organization
- Power BI can easily discover all layers
- Easier to manage permissions

**Future Improvement:**
When Fabric improves schema discovery, consider organizing by business domain:
- `mfflakehouse.sales_analytics.*`
- `mfflakehouse.customer_analytics.*`
- `mfflakehouse.product_analytics.*`

---

## Notebook Naming Convention

### Notebook File Naming Format

**Format:** `<phase>_<entity>_<purpose>.ipynb`

**Pattern Breakdown:**
- `<phase>` - Phase number (01, 02, 03, etc.)
- `<entity>` - Entity being processed (sessions, orders, products, etc.)
- `<purpose>` - What the notebook does (validate, transform, analyze, etc.)

### Examples

**Validation Notebooks (Phase 5):**
- `05_sessions_validate.ipynb` - Validates stg_sessions
- `05_orders_validate.ipynb` - Validates stg_orders
- `05_orchestration_status_check.ipynb` - Aggregates validation results

**Transformation Notebooks (Phase 6):**
- `06_sessions_transform.ipynb` - Transforms stg_sessions → trn_sessions
- `06_orders_transform.ipynb` - Transforms stg_orders → trn_orders
- `06_pageviews_transform.ipynb` - Transforms stg_website_pageviews → trn_website_pageviews

**Aggregation Notebooks (Phase 7):**
- `07_sessions_aggregate.ipynb` - Creates gold/fact tables from trn_sessions
- `07_orders_aggregate.ipynb` - Creates fact_orders from trn_orders
- `07_dimensions_create.ipynb` - Creates dimension tables

---

## Folder Structure Naming

### Repository Folder Organization

```
maven-fuzzy-factory-analytics/
├── docs/
│   ├── phase-4-data-quality/
│   ├── phase-5-orchestration/
│   ├── phase-6-transformation/
│   └── phase-7-gold-layer/
├── notebooks/
│   ├── 00-setup/
│   ├── 05-orchestration/
│   ├── 06-transformation/
│   └── 07-aggregation/
└── data/
    ├── data-dictionary.csv
    └── schemas/
```

### Folder Naming Rules

- Use kebab-case (lowercase with hyphens) for folder names
- Include phase number in folder name: `phase-5-orchestration`
- Use descriptive names that reflect layer or purpose
- Organize chronologically by phase number

---

## Dataflow Naming Convention

### Dataflow Naming Format

**Format:** `df_<entity>_<operation>`

Examples:
- `df_sessions_load` - Load sessions from CSV
- `df_orders_transform` - Transform orders
- `df_products_validate` - Validate products

### Rules

- Use `df_` prefix to distinguish from table names
- Include the main entity
- Describe the operation (load, transform, validate, aggregate)
- Use snake_case

---

## Pipeline Naming Convention

### Data Pipeline Naming Format

**Format:** `PL_<Purpose>_<Scope>`

Examples:
- `PL_Master_Data_Quality_Validation` - Main validation pipeline
- `PL_Daily_Data_Refresh` - Daily data refresh
- `PL_Gold_Layer_Aggregation` - Create gold layer tables

### Rules

- Use `PL_` prefix (Power Fx Pipeline)
- Use PascalCase (first letter capitalized per word)
- Be descriptive: include purpose and scope
- Avoid abbreviations (use "Data" not "D8a")

---

## Reference Table Naming

### Special Naming for Reference/Dimension Data

Reference tables or lookups use a `ref_` prefix:

Examples:
- `ref_data_dictionary` - Column definitions
- `ref_product_categories` - Product category lookup
- `ref_marketing_channels` - Marketing channel codes
- `ref_date_dimension` - Pre-built date table

### Rules

- Use `ref_` prefix for reference/lookup tables
- Naming describes the content
- Not typically refreshed frequently
- Smaller, static data sets

---

## Naming Convention Exceptions

### When to Break the Rules

**Quality Log Tables (Phase 5):**
- `data_quality_log` - Detailed check results (exception to `val_*` naming)
- `data_quality_summary` - Aggregated metrics (exception to `val_*` naming)

**Reasoning:**
- These are operational metadata, not data entities
- Universally understood names improve clarity
- Will be widely referenced across notebooks

---

## Entity Naming Guidelines

### Standard Entity Names (Singular)

Use singular entity names, not plural:

| Use | Don't Use | Reason |
|-----|-----------|--------|
| `session` | `sessions` | SQL standard practice |
| `order` | `orders` | Standard naming convention |
| `product` | `products` | Consistency |
| `pageview` | `pageviews` | Entity-based thinking |
| `refund` | `refunds` | Single entity per record |

### Compound Entity Names

For multi-word entities, use full descriptive names:

| Use | Don't Use | Reason |
|-----|-----------|--------|
| `order_item` | `oi` or `oitem` | Clear and descriptive |
| `website_pageview` | `web_pv` | Avoid abbreviations |
| `marketing_channel` | `mkt_ch` | Full names aid discovery |

---

## Documentation Examples

### Phase 6 Transformation Example

**Staging Table:** `stg_sessions`
↓ (Phase 5 validation)
**Validated Table:** `val_sessions`
↓ (Phase 6 transformation)
**Transformed Table:** `trn_sessions`

**Column Examples:**
- `stg_sessions.website_session_id` (raw column name)
- `trn_sessions.session_id` (normalized)
- `trn_sessions.is_repeat_session` (renamed from repeat_session)
- `trn_sessions.is_mobile` (derived: device_type = 'mobile')
- `trn_sessions.session_date` (derived from created_at)

---

## Implementation Checklist

Use these guidelines when:

- [ ] Creating new tables (apply appropriate layer prefix)
- [ ] Defining new columns (use standard suffixes)
- [ ] Creating notebooks (use phase + entity + purpose format)
- [ ] Setting up pipelines (use descriptive names)
- [ ] Organizing folders (kebab-case with phase numbers)
- [ ] Documenting transformations (reference this guide)

---

## Future-Proofing

### When to Revise This Document

- After Phase 7 completion (evaluate naming effectiveness)
- When new entity types are added
- If Fabric adds new schema capabilities
- When team size grows (consistency becomes more important)

### Suggested Review Date

**Review: After Phase 7 (December 2025)**

Assess whether naming convention is:
- Easy to understand for new team members
- Sufficient for discovering related tables
- Aligned with Power BI dashboards and reports
- Flexible for future expansion

---

## Quick Reference

| Layer | Prefix | Purpose | Example |
|-------|--------|---------|---------|
| Raw | `stg_` | As-is from source | `stg_orders` |
| Validated | `val_` | After quality checks | `val_orders` |
| Transformed | `trn_` | Business logic applied | `trn_orders` |
| Gold/Fact | `fct_` | Analytics-ready facts | `fct_orders` |
| Gold/Dimension | `dim_` | Dimension/lookup | `dim_products` |
| Gold/Mart | `mart_` | Domain-specific | `mart_sales` |
| Reference | `ref_` | Static lookups | `ref_data_dictionary` |

---

**Document Created:** November 26, 2025  
**Status:** Active - Use for Phase 6 and beyond  
**Last Updated:** November 26, 2025  
**Owner:** Data Engineering Team
