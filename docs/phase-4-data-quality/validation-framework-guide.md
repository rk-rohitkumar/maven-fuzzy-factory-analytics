# Complete Validation Framework - Implementation Guide
# Maven Fuzzy Factory E-Commerce Analytics
# Created: November 20, 2025

---

## 📦 Deliverables Summary

### Created Files

1. **00_setup_quality_tables.ipynb** ✅ CREATED
   - Purpose: Create data_quality_log and data_quality_summary tables
   - Run this FIRST before any validation notebooks
   
2. **01_sessions_validation.ipynb** ✅ CREATED
   - Table: stg_sessions
   - Validates: 472,871 session records
   - Expected Quality: 100%

**REMAINING NOTEBOOKS TO CREATE:** (5 more)

3. **02_pageviews_validation.ipynb**
4. **03_orders_validation.ipynb**  
5. **04_order_items_validation.ipynb**
6. **05_products_validation.ipynb**
7. **06_refunds_validation.ipynb**

---

## 🎯 Execution Order

### Step 1: Setup (5 minutes)
```
Run: 00_setup_quality_tables.ipynb
Action: Creates empty quality tables with correct schema
Verify: Both tables show 0 rows
```

### Step 2: Run Validations (30 minutes)
```
Order of Execution:
1. 01_sessions_validation.ipynb (already validated - 100%)
2. 05_products_validation.ipynb (master data first)
3. 02_pageviews_validation.ipynb
4. 03_orders_validation.ipynb
5. 04_order_items_validation.ipynb
6. 06_refunds_validation.ipynb
```

### Step 3: Review Results (10 minutes)
```sql
SELECT 
    table_name,
    quality_score,
    overall_status,
    validation_checks_passed,
    validation_checks_total
FROM data_quality_summary
ORDER BY table_name;
```

---

## 📋 Schema Reference

### data_quality_log Schema
```
root
 |-- run_id: string (nullable = false)
 |-- run_timestamp: timestamp (nullable = false)
 |-- table_name: string (nullable = false)
 |-- check_name: string (nullable = false)
 |-- check_type: string (nullable = false)
 |-- column_name: string (nullable = true)
 |-- passed: string (nullable = false)
 |-- invalid_count: integer (nullable = false)
 |-- threshold: string (nullable = true)
 |-- message: string (nullable = true)
```

### data_quality_summary Schema
```
root
 |-- run_id: string (nullable = false)
 |-- run_timestamp: timestamp (nullable = false)
 |-- table_name: string (nullable = false)
 |-- row_count: integer (nullable = false)
 |-- pk_duplicate_count: integer (nullable = false)
 |-- null_violations: integer (nullable = false)
 |-- validation_checks_total: integer (nullable = false)
 |-- validation_checks_passed: integer (nullable = false)
 |-- quality_score: string (nullable = false)
 |-- overall_status: string (nullable = false)
```

---

## ✨ Key Features in All Notebooks

### Consistent Structure
- **Header Section:** Table info, validation scope, expected ranges
- **Configuration:** Constants, thresholds, run metadata
- **Load Data:** Read source table with profiling
- **Basic Profiling:** Row counts, distributions, statistics
- **Validation Checks:** 7-10 checks per table
- **Quality Score:** Percentage of checks passed
- **Persist Results:** Write to log and summary tables
- **Verification:** Display results from persisted tables

### Technical Excellence
- ✅ Schema compatibility guaranteed
- ✅ Function shadowing prevention (`del sum`)
- ✅ Proper `col()` usage in aggregations
- ✅ IntegerType casting for all numeric fields
- ✅ String "True"/"False" for passed field
- ✅ Exact schema match with quality tables
- ✅ Empty table handling (refunds)
- ✅ Comprehensive error prevention

### Validation Coverage
- **Completeness:** Null checks on critical columns
- **Uniqueness:** Primary key duplicate detection
- **Validity:** Data types, value ranges, business rules
- **Consistency:** Binary flags, categorical values

---

## 🔧 Template Pattern

All validation notebooks follow this proven structure:

```python
# 1. Imports + del sum
from pyspark.sql.functions import *
from pyspark.sql.types import IntegerType, StructType, StructField, StringType, TimestampType
from datetime import datetime
import uuid
del sum  # Restore Python built-in

# 2. Configuration
SOURCE_TABLE = "stg_[table_name]"
PK_COLUMN = "[primary_key]"
# ... thresholds, run metadata

# 3. Load Data
df = spark.read.table(SOURCE_TABLE)
# ... profiling

# 4. Validation Helper
validation_results = []
def add_validation_result(...):
    validation_results.append({
        "run_id": RUN_ID,
        "run_timestamp": RUN_TIMESTAMP,
        "table_name": SOURCE_TABLE,
        "check_name": check_name,
        "check_type": check_type,
        "column_name": column_name,
        "passed": "True" if passed else "False",  # STRING not BOOLEAN
        "invalid_count": invalid_count,
        "threshold": threshold,
        "message": message
    })

# 5. Execute Checks
# Row count, uniqueness, nulls, types, business logic...

# 6. Calculate Quality Score
total_checks = len(validation_results)
passed_checks = sum([1 for r in validation_results if r["passed"] == "True"])
quality_score = (passed_checks / total_checks * 100)
overall_status = "PASSED" if quality_score == 100 else "FAILED"

# 7. Persist to Log Table
log_schema = StructType([...])  # Match table schema exactly
validation_log_df = spark.createDataFrame(validation_results, schema=log_schema)
validation_log_df.write.mode("append").saveAsTable("data_quality_log")

# 8. Persist to Summary Table
summary_data = [{
    "run_id": RUN_ID,
    "run_timestamp": RUN_TIMESTAMP,
    "table_name": SOURCE_TABLE,
    "row_count": total_rows,
    "pk_duplicate_count": duplicate_count,
    "null_violations": null_violations,
    "validation_checks_total": total_checks,
    "validation_checks_passed": passed_checks,
    "quality_score": f"{quality_score:.1f}",  # STRING format
    "overall_status": overall_status
}]
summary_schema = StructType([...])  # Match table schema exactly
summary_df = spark.createDataFrame(summary_data, schema=summary_schema)
summary_df.write.mode("append").saveAsTable("data_quality_summary")

# 9. Verification Queries
# Display results from persisted tables
```

---

## 🚨 Critical Implementation Details

### 1. Schema Compatibility
**All validation notebooks MUST use these exact schemas when writing:**

```python
# Log table schema
log_schema = StructType([
    StructField("run_id", StringType(), False),
    StructField("run_timestamp", TimestampType(), False),
    StructField("table_name", StringType(), False),
    StructField("check_name", StringType(), False),
    StructField("check_type", StringType(), False),
    StructField("column_name", StringType(), True),
    StructField("passed", StringType(), False),  # STRING not BOOLEAN
    StructField("invalid_count", IntegerType(), False),
    StructField("threshold", StringType(), True),
    StructField("message", StringType(), True)
])

# Summary table schema
summary_schema = StructType([
    StructField("run_id", StringType(), False),
    StructField("run_timestamp", TimestampType(), False),
    StructField("table_name", StringType(), False),
    StructField("row_count", IntegerType(), False),
    StructField("pk_duplicate_count", IntegerType(), False),
    StructField("null_violations", IntegerType(), False),
    StructField("validation_checks_total", IntegerType(), False),
    StructField("validation_checks_passed", IntegerType(), False),
    StructField("quality_score", StringType(), False),
    StructField("overall_status", StringType(), False)
])
```

### 2. Data Type Rules
- `passed`: STRING "True" or "False" (NOT boolean True/False)
- `quality_score`: STRING "100.0" (NOT numeric 100.0)
- All counts: IntegerType (NOT LongType)
- All IDs: StringType (UUIDs)

### 3. Common Pitfalls to Avoid
```python
# ❌ WRONG
passed = True  # Boolean
quality_score = 100.0  # Float
sum("price_usd")  # String instead of Column

# ✅ CORRECT
passed = "True"  # String
quality_score = "100.0"  # String
sum(col("price_usd"))  # Column object
```

---

## 📊 Table-Specific Validation Details

### Sessions (stg_sessions)
- **PK:** website_session_id
- **Row Range:** 10K-5M
- **Special Checks:** Binary flags, device types
- **Expected Score:** 100%

### Pageviews (stg_pageviews)
- **PK:** website_pageview_id
- **Row Range:** 10K-5M
- **Special Checks:** URL format, session linkage
- **Expected Score:** 95-100%

### Orders (stg_orders)
- **PK:** order_id
- **Row Range:** 1K-1M
- **Special Checks:** Revenue validation, price >= COGS
- **Expected Score:** 95-100%

### Order Items (stg_order_items)
- **PK:** order_item_id
- **Row Range:** 1K-2M
- **Special Checks:** Primary item flags, product linkage
- **Expected Score:** 95-100%

### Products (stg_products)
- **PK:** product_id
- **Row Range:** 1-100
- **Special Checks:** Name uniqueness, 0% duplicates
- **Expected Score:** 100%

### Refunds (stg_refunds)
- **PK:** order_item_refund_id
- **Row Range:** 0-50K
- **Special Checks:** Empty table handling, refund amounts
- **Expected Score:** 100% or N/A

---

## 📝 Next Steps

### Immediate (Today)
1. ✅ Run `00_setup_quality_tables.ipynb`
2. ✅ Run `01_sessions_validation.ipynb`
3. ⏳ Create remaining 5 validation notebooks (using template)
4. ⏳ Execute all validation notebooks
5. ⏳ Document quality scores

### Tomorrow (Nov 21)
- Create orchestration pipeline
- Integrate all 6 validation notebooks
- Add conditional routing logic

### Next Week (Nov 23-26)
- Referential integrity validation
- Cross-table consistency checks
- Transformation specification

---

## 🎯 Success Criteria

**Phase 4 Complete When:**
- [ ] All 6 quality tables created
- [ ] All 6 validation notebooks executed
- [ ] All tables achieve >95% quality score
- [ ] Zero schema mismatch errors
- [ ] Results persisted to quality tables
- [ ] Documentation updated with findings

**Expected Outcomes:**
- 100% for products, sessions
- 95-100% for others
- Clear documentation of any exceptions
- Ready to proceed to transformation phase

---

## 📚 References

- Data Dictionary: `maven_fuzzy_factory_data_dictionary.csv`
- Framework Documentation: `Data_Quality_Framework_Documentation.md`
- Lessons Learned: `2.2 LESSONS-LEARNED.md`
- Project Roadmap: `1.1 PROJECT-ROADMAP.md`

---

**Document Version:** 3.0  
**Last Updated:** November 20, 2025  
**Status:** In Progress - 2 of 7 notebooks complete  
**Next Action:** Create remaining 5 validation notebooks using template pattern