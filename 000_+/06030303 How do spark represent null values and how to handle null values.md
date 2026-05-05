---
up:
  - "[[000_+/060303 Advance operator|060303 Advance operator]]"
down:
prev:
topic: false
question: How do spark represent null values and how to handle null values?
---
# How do spark represent null values and how to handle null values?


> [!Summary] Summary
> Contents


|**Goal**|**Best Method**|**Why?**|
|---|---|---|
|**Cleanliness**|`na.drop()`|Removes noise if the record is useless without that data.|
|**Data Retention**|`na.fill()`|Keeps the row but provides a placeholder (common in ML).|
|**Priority Selection**|`coalesce()`|Efficiently merges multiple source columns into one.|
|**Filtering**|`filter(col.isNull())`|Best for auditing or finding the rows that need fixing.|


Spark handles nulls at two levels: the schema level and the physical storage level.

### 1. The Schema (`nullable` flag)
Every column in a Spark DataFrame has a metadata property called `nullable`.
- If `nullable=true`, the column can contain null values.
- If `nullable=false`, Spark assumes the column is always populated.

### 2. Physical Representation (Tungsten)
Internally, Spark uses its **Tungsten** execution engine to manage memory efficiently. Instead of storing a special "null object," Spark uses a **Null Bitmask** (or Null Bitmap) for each row.
- A bitmask is a small sequence of bits where each bit corresponds to a column.
- If the $3^{rd}$ bit is `1`, the $3^{rd}$ column in that row is `null`.
- This allows Spark to store data compactly without wasting space on missing values.

## How to Handle Null Values

PySpark provides the `DataFrame.na` (DataFrameNaFunctions) sub-module, which is the primary toolkit for dealing with missing data.
### 1. Dropping Nulls (`df.na.drop`)
You can remove rows containing nulls based on specific criteria.

```python
# Drop any row that contains at least one null
df.na.drop()

# Drop rows only if ALL values are null
df.na.drop(how="all")

# Drop rows only if nulls appear in specific columns
df.na.drop(subset=["email", "phone"])
```

### 2. Filling/Imputing Nulls (`df.na.fill`)
Replacing nulls with a "safe" default value is common in data cleaning.
```python
# Fill all nulls in numeric columns with 0
df.na.fill(value=0)

# Fill nulls with specific values for specific columns
df.na.fill({"Age": 25, "City": "Unknown"})
```

### 3. The `coalesce` Function
The `coalesce()` function is incredibly useful when you have multiple columns and you want to pick the first one that **isn't** null.

```python
from pyspark.sql import functions as F

# If 'Phone' is null, take 'Mobile'; if both are null, take 'Work_Phone'
df.withColumn("Contact", F.coalesce(F.col("Phone"), F.col("Mobile"), F.col("Work_Phone")))
```

### 4. Conditional Handling (`when` / `otherwise`)

If you need more complex logic (e.g., "if null, do calculation A, otherwise do B"), use `when`.

```python
df.withColumn("Status", 
    F.when(F.col("Score").isNull(), "Not Tested")
     .otherwise(F.col("Score"))
)
```


### A Quick Warning on Math

In Spark SQL, most arithmetic operations involving a `null` result in `null`. For example, `5 + null` will always be `null`. If you are performing aggregations like `sum()`, Spark will ignore the nulls rather than failing, but for row-level math, you should always handle nulls first!

Are you dealing with a dataset where you need to fill nulls based on a statistical measure, like the mean or median of a column?