# Phase 6: Transformation Layer - Planning & Design

**Project:** Maven Fuzzy Factory E-Commerce Analytics  
**Date:** November 25, 2025  
**Phase:** 6 (Data Transformation Implementation)  
**Duration:** 10-14 days  

---

## Overview

Phase 6 focuses on transforming staging tables into clean, business-ready intermediate tables. This phase builds directly on Phase 5's validated data and prepares data for Phase 7's gold layer aggregation and Phase 8's Power BI dashboards.

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

## Transformation Specifications by Table

### 1. stg_sessions → stg_sessions_transformed

**Key Transformations:**
- Remove duplicate session IDs (keep earliest)
- Standardize UTM fields (null → 'Direct')
- Convert device_type to uppercase
- Add derived fields:
  - is_mobile (boolean from device_type)
  - session_year, session_month (from created_at)

**Business Rules:**
- Keep only sessions with valid user_id
- Exclude sessions older than 2010
- Convert repeat_session to explicit boolean

---

### 2. stg_website_pageviews → stg_pageviews_transformed

**Key Transformations:**
- Remove duplicate page_view_id records
- Validate session_id exists in sessions table
- Add page_sequence (row number within session)
- Extract URL components (path, query params)

**Business Rules:**
- Only include pageviews for valid sessions
- Standardize URL format
- Calculate time_on_page where possible

---

### 3. stg_orders → stg_orders_transformed

**Key Transformations:**
- Remove duplicate order_id records
- Validate user_id exists in sessions
- Add order_year, order_month
- Flag first_order per user

**Business Rules:**
- Only include orders with valid user_id
- Exclude test orders (if identifiable)
- Calculate order_value from order_items

---

### 4. stg_order_items → stg_orderitems_transformed

**Key Transformations:**
- Remove duplicate order_item_id records
- Validate order_id and product_id exist
- Calculate line_total (qty × unit_price)
- Add product category/name from products table

**Business Rules:**
- Only include items for valid orders
- Validate pricing > 0
- Handle refunds separately

---

### 5. stg_products → stg_products_transformed

**Key Transformations:**
- Remove duplicate product_id records
- Standardize product_name (trim, proper case)
- Validate price > 0
- Add product_category groupings

**Business Rules:**
- Only include active products
- Maintain product catalog consistency
- Track price changes over time

---

### 6. stg_order_item_refunds → stg_refunds_transformed

**Key Transformations:**
- Remove duplicate refund_id records
- Validate order_item_id exists
- Standardize refund_status
- Link to original order details

**Business Rules:**
- Only include refunds for valid order items
- Map refund status to standard values
- Calculate refund_amount correctly

---

## Implementation Approach

### Option A: Dataflows (Recommended for GUI)
- Use Power Query in Fabric Dataflows
- Visual transformation interface
- No-code/low-code approach
- Easy version control with Git integration

### Option B: PySpark Notebooks (Recommended for Code)
- Full control over transformation logic
- Reusable code patterns
- Easier testing and debugging
- Better for complex business logic

### Option C: Hybrid (Recommended for Production)
- Use Dataflows for simple transformations
- Use Notebooks for complex business logic
- Combine in pipeline for orchestration

---

## Success Criteria

- [ ] All 6 tables transformed without data loss
- [ ] 0 duplicates in output tables
- [ ] 0 null values in critical fields
- [ ] All foreign key relationships valid
- [ ] Transformation logic documented
- [ ] Derived metrics calculated correctly
- [ ] Output tables ready for gold layer
- [ ] Pipeline execution time < 15 minutes

---

## Next Steps

1. **Choose transformation approach** (A, B, or C)
2. **Create transformation notebooks/dataflows** (2-3 days)
3. **Test with sample data** (1 day)
4. **Integrate into pipeline** (1 day)
5. **Validate full pipeline** (1 day)
6. **Document and review** (1 day)

---

**Owner:** Data Engineering Team  
**Created:** November 25, 2025
