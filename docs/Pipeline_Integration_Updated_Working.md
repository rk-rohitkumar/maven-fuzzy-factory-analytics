# Pipeline Integration: Updated with Working Implementations
# Based on Successful Sessions Validation Notebook
# November 16, 2025

## KEY IMPLEMENTATION FIXES

### 1. Schema Casting for Delta Lake Compatibility

**CRITICAL:** PySpark creates `long` (64-bit) integers by default, but Delta tables may expect `int` (32-bit).

#### Working Code Pattern:

```python
from pyspark.sql.functions import col
from pyspark.sql.types import IntegerType

# Cast log table integer columns
validation_log_df = validation_log_df.withColumn("invalid_count", col("invalid_count").cast(IntegerType()))

# Cast summary table integer columns
summary_df = summary_df.withColumn("row_count", col("row_count").cast(IntegerType()))                        .withColumn("pk_duplicate_count", col("pk_duplicate_count").cast(IntegerType()))                        .withColumn("null_violations", col("null_violations").cast(IntegerType()))                        .withColumn("validation_checks_total", col("validation_checks_total").cast(IntegerType()))                        .withColumn("validation_checks_passed", col("validation_checks_passed").cast(IntegerType()))

# Now safe to write
summary_df.write.mode("append").option("mergeSchema", "true").format("delta").saveAsTable("mff_lakehouse.dbo.data_quality_summary")
```

### 2. Python's sum() vs Spark's sum()

**CRITICAL:** Do NOT import sum from pyspark.sql.functions

#### Working Code Pattern:

```python
# At top of notebook after imports:
from pyspark.sql.functions import *
del sum  # DELETE Spark's sum to restore Python's built-in

# Now this works correctly:
passed_checks = sum(1 for check in validation_checks if check["passed"])
```

---

## Working Notebook Code Sections

### Section A: Configuration & Imports

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import *
del sum  # CRITICAL: Remove Spark sum
from pyspark.sql.types import IntegerType  # For casting
from datetime import datetime
import json

# Configuration
LAKEHOUSE_NAME = "mff_lakehouse.dbo"
SOURCE_TABLE = "stg_sessions"
RUN_TIMESTAMP = datetime.now()
RUN_ID = f"sessions_{RUN_TIMESTAMP.strftime('%Y%m%d_%H%M%S')}"

THRESHOLDS = {
    "max_duplicate_rate": 0.0,
    "max_null_rate_critical": 0.0,
    "max_null_rate_optional": 0.3,
    "min_row_count": 10000,
    "max_row_count": 5000000,
}
```

### Section B: Quality Score Calculation

```python
# Use Python sum, not Spark sum
total_checks = len(validation_checks)
passed_checks = sum(1 for check in validation_checks if check["passed"])
quality_score = (passed_checks / total_checks * 100) if total_checks > 0 else 0

row_count_valid = THRESHOLDS["min_row_count"] <= row_count <= THRESHOLDS["max_row_count"]
overall_status = "PASSED" if quality_score >= 98 and row_count_valid and pk_duplicate_rate <= THRESHOLDS["max_duplicate_rate"] else "FAILED"
```

### Section C: Write Validation Log with Casting

```python
from pyspark.sql.types import IntegerType

validation_log_records = []
for check in validation_checks:
    validation_log_records.append({
        "run_id": RUN_ID,
        "run_timestamp": RUN_TIMESTAMP,
        "table_name": SOURCE_TABLE,
        "check_name": check["check_name"],
        "check_passed": check["passed"],
        "invalid_count": check["invalid_count"],
        "message": check["message"]
    })

validation_log_df = spark.createDataFrame(validation_log_records)

# CAST invalid_count to match table schema
validation_log_df = validation_log_df.withColumn("invalid_count", col("invalid_count").cast(IntegerType()))

# Write to table
validation_log_df.write.mode("append").option("mergeSchema", "true").format("delta").saveAsTable("mff_lakehouse.dbo.data_quality_log")
print(f"✓ Written {len(validation_log_records)} validation checks to data_quality_log")
```

### Section D: Write Summary with Casting

```python
summary_record = {
    "run_id": RUN_ID,
    "run_timestamp": RUN_TIMESTAMP,
    "table_name": SOURCE_TABLE,
    "row_count": row_count,
    "pk_duplicate_count": pk_duplicate_count,
    "pk_duplicate_rate": pk_duplicate_rate,
    "null_violations": len(null_violations),
    "validation_checks_total": total_checks,
    "validation_checks_passed": passed_checks,
    "quality_score": quality_score,
    "overall_status": overall_status,
    "min_date": str(date_stats.min_date),
    "max_date": str(date_stats.max_date)
}

summary_df = spark.createDataFrame([summary_record])

# CAST all integer columns to match table schema
summary_df = summary_df.withColumn("row_count", col("row_count").cast(IntegerType()))                        .withColumn("pk_duplicate_count", col("pk_duplicate_count").cast(IntegerType()))                        .withColumn("null_violations", col("null_violations").cast(IntegerType()))                        .withColumn("validation_checks_total", col("validation_checks_total").cast(IntegerType()))                        .withColumn("validation_checks_passed", col("validation_checks_passed").cast(IntegerType()))

# Write to table
summary_df.write.mode("append").option("mergeSchema", "true").format("delta").saveAsTable("mff_lakehouse.dbo.data_quality_summary")
print(f"✓ Written summary record to data_quality_summary")
```

---

## Successful Execution Results

### Sessions Table Validation (Nov 16, 2025)

```
Quality Score: 100.0%
Status: PASSED
Row Count: 472,871 ✓
Duplicates: 0 (0.00%) ✓
Validation Checks: 6/6 PASSED ✓

Data Written To:
- data_quality_log: 6 records ✓
- data_quality_summary: 1 record ✓
```

---

## Troubleshooting Guide

### Error: DELTA_FAILED_TO_MERGE_FIELDS

**Cause:** DataFrame column type doesn't match Delta table schema

**Fix:** Add explicit casting before write:
```python
df = df.withColumn("column_name", col("column_name").cast(IntegerType()))
```

### Error: PySparkTypeError - Argument col should be a Column or str, got generator

**Cause:** Spark's sum function imported, replacing Python's sum

**Fix:** Delete Spark's sum early in notebook:
```python
from pyspark.sql.functions import *
del sum  # Restore Python's built-in sum
```

### Error: Artifact not found: mff_lakehouse.dbo

**Cause:** Schema doesn't exist in lakehouse

**Fix:** Create schema or remove schema from table name:
```python
# Option 1: Create schema
spark.sql("CREATE SCHEMA IF NOT EXISTS mff_lakehouse.dbo")

# Option 2: Use lakehouse directly
.saveAsTable("mff_lakehouse.data_quality_log")
```

---

## Ready for Remaining Tables

This working pattern can be applied to all 6 tables:
1. ✓ stg_sessions - COMPLETED
2. ⏱ stg_pageviews - READY
3. ⏱ stg_orders - READY
4. ⏱ stg_order_items - READY
5. ⏱ stg_products - READY
6. ⏱ stg_refunds - READY

Simply update:
- SOURCE_TABLE variable
- Table-specific validation checks
- Quality thresholds per table

All code patterns proven and tested ✓

