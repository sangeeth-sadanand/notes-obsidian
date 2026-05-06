---
up:
  - "[[003_skills/data-engineering/060303 Advance operator|060303 Advance operator]]"
down:
prev:
topic: false
question: Various methods to unpivot dataframes
---
# Various methods to unpivot dataframes


> [!Summary] Summary
> Unpivoting (often called **melting**) is the inverse of pivoting. It takes a "wide" dataset with multiple columns and collapses them into "long" rows, usually resulting in two new columns: one for the **header names** (labels) and one for the **values**.
> 
> |**Method**|**Spark Version**|**Best For...**|
> |---|---|---|
> |**`unpivot()`**|3.4+|**General use.** Cleanest syntax and natively optimized.|
> |**`stack()`**|All|**Legacy environments** or high-performance SQL-heavy pipelines.|
> |**`explode()`**|All|**Complex types.** Useful if you are already working with nested structures.|
> |**`melt()`**|3.2+|**Data Scientists** who prefer the Pandas syntax style.|
> 

Unpivoting (often called **melting**) is the inverse of pivoting. It takes a "wide" dataset with multiple columns and collapses them into "long" rows, usually resulting in two new columns: one for the **header names** (labels) and one for the **values**.

### 1. The Native `unpivot` Method (Spark 3.4+)
If you are using Spark 3.4 or later, there is a dedicated `unpivot` function. This is the most readable and efficient way to handle the transformation.
```python
# df has columns: 'ID', '2023_Sales', '2024_Sales'
unpivoted_df = df.unpivot(
    ids="ID", 
    values=["2023_Sales", "2024_Sales"], 
    variableColumnName="Year", 
    valueColumnName="Sales"
)
```
- **ids**: Columns to keep as-is (the anchor).
- **values**: Columns to "melt" down into rows.
- **variableColumnName**: The name for the new column containing the old headers.
- **valueColumnName**: The name for the new column containing the data.
    

### 2. Using the `stack` Expression
For versions of Spark older than 3.4, the `stack` function within `selectExpr` is the industry standard. It manually "stacks" columns on top of each other.

```python
# Syntax: stack(n, 'label1', col1, 'label2', col2, ...)
n = 2 # Number of columns to unpivot
unpivoted_df = df.selectExpr(
    "ID", 
    f"stack({n}, '2023', 2023_Sales, '2024', 2024_Sales) as (Year, Sales)"
)
```

- **Pros:** Highly performant and works on all Spark versions.
- **Cons:** Can be tedious to write if you have dozens of columns (requires dynamic string building).
    

### 3. The `explode` and `array` Approach

This method is useful if you want to avoid raw SQL strings. You create an array of structs containing the column name and value, then "explode" that array into separate rows.
```python
from pyspark.sql import functions as F

unpivoted_df = df.withColumn("kv", F.explode(F.array(
    F.struct(F.lit("2023").alias("Year"), F.col("2023_Sales").alias("Sales")),
    F.struct(F.lit("2024").alias("Year"), F.col("2024_Sales").alias("Sales"))
))).select("ID", "kv.Year", "kv.Sales")
```
- **Pros:** Entirely programmatic; no string manipulation required.
- **Cons:** More verbose than other methods.

### 4. Pandas API on Spark (`melt`)
If you are coming from a Data Science background, you can use the Pandas-on-Spark API (formerly Koalas) which provides a familiar `melt` function.

```python
import pyspark.pandas as ps

# Convert Spark DF to Pandas-on-Spark DF
ps_df = df.pandas_api()

unpivoted_ps_df = ps_df.melt(
    id_vars=['ID'], 
    value_vars=['2023_Sales', '2024_Sales'],
    var_name='Year', 
    value_name='Sales'
)

# Convert back to standard PySpark DF if needed
unpivoted_df = unpivoted_ps_df.to_spark()
```

- **Pros:** Extremely intuitive for Pandas users.
- **Cons:** Involves overhead when switching between the standard Spark API and the Pandas-on-Spark API.

