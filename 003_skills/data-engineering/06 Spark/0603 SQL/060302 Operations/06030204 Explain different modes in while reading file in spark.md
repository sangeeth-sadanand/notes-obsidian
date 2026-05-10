---
up:
  - "[[003_skills/data-engineering/06 Spark/0603 SQL/060302 Operations/060302 Operations|060302 Operations]]"
down:
prev:
topic: false
question: Explain different modes in while reading file in spark
---
# Explain different modes in while reading file in spark


> [!Summary] Summary
> - We can use modes to handle corrupt or malformed records 
> - We can set mode using `.option("mode", "mode-name")` 
> - **Permissive**: In this mode, if there is mismatch in the schema provided and the data then the data is replaced by null 
> - **Drop Malformed**: In this mode the malformed row, i. e row with mismatch schema are dropped 
> - **Fast Fail:**- In this mode, if any data mismatches with the provided schema then it throws error which stops execution.


When reading data from structured sources like CSV or JSON, Spark provides **read modes** to handle "corrupt" or "malformed" records. Since real-world data is often messy (missing columns, extra delimiters, or wrong data types), these modes allow you to define the fault-tolerance of your pipeline.
You set the mode using `.option("mode", "MODE_NAME")`.

## 1. Permissive (Default)
The `PERMISSIVE` mode is the most "forgiving." It tries to rescue as much data as possible.
- **Behavior:** When it encounters a corrupt record, it sets all fields to `null` for that row.
- **The Special Column:** If you define a specific column in your schema called `_corrupt_record`, Spark will put the entire malformed raw string into that column so you can audit it later.
- **Use Case:** Ideal when you don't want your entire job to crash just because one line in a million is bad.

```python
df = spark.read.format("csv") \
    .option("mode", "PERMISSIVE") \
    .option("columnNameOfCorruptRecord", "_corrupt_record") \
    .load("data.csv")
```

## 2. DropMalformed
This mode is "selective." It ignores the bad parts of your dataset and moves on.
- **Behavior:** It silently drops any row that contains a malformed record or doesn't match the schema.
- **Result:** The resulting DataFrame will only contain "clean" rows.
- **Use Case:** Good for data exploration or when you know that a small percentage of bad data won't affect your final analysis/metrics.
    
```python
df = spark.read.format("json") \
    .option("mode", "DROPMALFORMED") \
    .load("data.json")
```

## 3. FailFast
This mode is "strict." It demands perfection from your input data.
- **Behavior:** As soon as Spark encounters a single malformed record, it throws an exception and stops the entire job immediately.
- **Use Case:** Critical for high-stakes financial or medical data where "guessing" or "dropping" data could lead to dangerous results. It’s the best way to ensure total data integrity.
```python
df = spark.read.format("csv") \
    .option("mode", "FAILFAST") \
    .load("sensitive_data.csv")
```

## Comparison of Read Modes

|**Mode**|**Action on Malformed Row**|**Job Status**|**Data Integrity**|
|---|---|---|---|
|**Permissive**|Sets fields to `null`|Continues|Medium (potential nulls)|
|**DropMalformed**|Deletes the row|Continues|High (but missing rows)|
|**FailFast**|Stops immediately|**Crashes**|**Highest**|
### Pro-Tip: The `_corrupt_record` Pattern

If you use `PERMISSIVE` mode, it's highly recommended to explicitly add the `_corrupt_record` field to your schema definition. Without it, you'll see nulls in your data but won't know _why_ the record failed.

```python
from pyspark.sql.types import *

schema = StructType([
    StructField("id", IntegerType(), True),
    StructField("name", StringType(), True),
    StructField("_corrupt_record", StringType(), True) # Captures the raw "bad" string
])
```

