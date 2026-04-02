## Pivot and Unpivot

### What Pivot Does

- **Definition:** Transforms distinct values of one column into new columns while applying an aggregation (e.g., `sum`, `avg`).
- **When to use:** Reporting, cross-tab summaries, or when you need one column per category. 

```python
data = [("East","Jan",100),("East","Feb",200),("West","Jan",300)]
df = spark.createDataFrame(data, ["Region","Month","Sales"])
pivot_df = df.groupBy("Region").pivot("Month").sum("Sales")
pivot_df.show()
```

**Result:** one row per `Region`, columns `Jan`, `Feb` with aggregated sales.
### What Unpivot (Melt) Does

- **Definition:** Converts wide-format columns into rows by producing a **variable** column (the former column name) and a **value** column (the cell value).
- **When to use:** Normalization, feeding ML pipelines, or when downstream code expects long format. 

```python
wide_df = spark.createDataFrame([("East",100,200),("West",300,None)], ["Region","Jan","Feb"])
unpivot_df = wide_df.unpivot(ids=["Region"], values=["Jan","Feb"], variableColumnName="Month", valueColumnName="Sales")
unpivot_df.show()
```

**Result:** rows like `(East, Jan, 100)`, `(East, Feb, 200)`, `(West, Jan, 300)`.
### What `stack` does 

- **Purpose:** **Unpivot** selected columns into multiple rows per original row.
- **Where used:** Inside `select()` as an expression or via `functions.stack`.
- **Behavior:** Produces new columns (by default `col0`, `col1`) from grouped column/value pairs. [Microsoft Learn](https://learn.microsoft.com/en-us/azure/databricks/pyspark/reference/functions/stack)

```python
from pyspark.sql import functions as F

# sample wide DataFrame
df = spark.createDataFrame([("Spain",101,201,301),("Italy",103,203,303)],
                           ["Country","2018","2019","2020"])

# unpivot years into (Year, CPI)
expr = "stack(3, '2018', `2018`, '2019', `2019`, '2020', `2020`) as (Year, CPI)"
unpivoted = df.select("Country", F.expr(expr))
unpivoted.show()
```

```python
from pyspark.sql import functions as F
unpivoted = df.select("Country", F.stack(F.lit(3), F.lit('2018'), df['2018'],
                                        F.lit('2019'), df['2019'],
                                        F.lit('2020'), df['2020']).alias("Year","CPI"))
```

**Result:** rows like `(Spain, 2018, 101)`, `(Spain, 2019, 201)`, etc. [Microsoft Learn](https://learn.microsoft.com/en-us/azure/databricks/pyspark/reference/functions/stack)

### How Spark Executes Them

- **Pivot:** usually causes a **shuffle** because Spark must group and aggregate by the pivot key; expensive on high-cardinality pivot columns. 
- **Unpivot:** is a local transformation (no aggregation) that expands rows; cost depends on 


## Handling Null

### 1. **fillna()**

- **Purpose:** Replace `NaN`/`null` values with a specified value.
    
    ```python
    df.fillna({'age': 0, 'city': 'Unknown'})
    ```
    
    - Replaces nulls in `age` with `0` and in `city` with `"Unknown"`.

### 2. **dropna()**

- **Purpose:** Remove rows (or columns) containing nulls.
    
    ```python
    df.dropna(how='any')   # drop rows with ANY null
    df.dropna(how='all')   # drop rows with ALL nulls
    df.dropna(subset=['age', 'city'])  # drop if null in specific columns
    ```
    
### 3. **replace()**

- **Purpose:** Replace specific values (including nulls if explicitly mentioned).

    ```python
    df.replace(['NA', 'null'], None)
    ```
    
    - Converts string placeholders into actual `None`.

### Quick Comparison Table

| Operation   | PySpark Example             | Use Case               |
| ----------- | --------------------------- | ---------------------- |
| **fillna**  | `df.fillna({'age':0})`      | Impute missing values  |
| **dropna**  | `df.dropna(subset=['age'])` | Remove incomplete rows |
| **replace** | `df.replace('NA', None)`    | Convert placeholders   |


## User defined function (UDF)

### 1. **What are UDFs?**

- **Definition:** Custom functions written in Python, registered with Spark SQL, and applied to DataFrame columns.
- **Purpose:** Extend Spark’s built-in functions when you need custom logic.
- **Execution:** Row-by-row, with serialization between JVM and Python (can be slower).

### 2. **Creating a UDF**

```python
from pyspark.sql.functions import udf
from pyspark.sql.types import IntegerType

# Python function
def square(x):
    return x * x if x is not None else None

# Register as UDF
square_udf = udf(square, IntegerType())

# Apply to DataFrame
df.withColumn("squared", square_udf(df["value"]))
```
### 3. **Registering UDFs for SQL**

```python
spark.udf.register("square_udf", square, IntegerType())

df.createOrReplaceTempView("table")
spark.sql("SELECT value, square_udf(value) AS squared FROM table").show()
```

### 4. **Limitations**

- **Performance:** Slower than built-in functions due to row-by-row serialization.
- **Optimization:** UDFs are black boxes to Spark’s optimizer (Catalyst), so they can’t be optimized like native functions.
- **Best Practice:** Use built-in functions whenever possible; fall back to UDFs only when necessary.

### 5. **Alternatives**

- **Pandas UDFs (Vectorized):** Faster, batch-based execution using Apache Arrow.
- **SQL Functions:** Prefer native Spark SQL functions for performance and optimization.


## Pandas UDF
### 1. **What are Pandas UDFs?**

- **Definition:** User Defined Functions that use **Apache Arrow** to transfer data in batches and apply **vectorized Pandas operations**.
- **Why:** They’re much faster than regular UDFs because they avoid row-by-row serialization.
- **Introduced in:** Spark 2.3 (with Arrow support).

### 2. **Types of Pandas UDFs**

| Type            | Input                                | Output           | Use Case                                |
| --------------- | ------------------------------------ | ---------------- | --------------------------------------- |
| **Scalar**      | Pandas Series                        | Pandas Series    | Element-wise transformations            |
| **Grouped Map** | Pandas DataFrame (per group)         | Pandas DataFrame | Custom group-level operations           |
| **Iterator**    | Iterator of Pandas Series/DataFrames | Iterator         | Streaming batches for memory efficiency |

### 3. **Scalar Pandas UDF**

```python
from pyspark.sql.functions import pandas_udf
from pyspark.sql.types import IntegerType
import pandas as pd

@pandas_udf(IntegerType())
def square(s: pd.Series) -> pd.Series:
    return s * s

df.withColumn("squared", square(df["value"]))
```

- Operates on **Pandas Series**.
- Best for vectorized math or string ops.

### 4. **Grouped Map Pandas UDF**

```python
from pyspark.sql.functions import pandas_udf, PandasUDFType

@pandas_udf("id long, mean_val double", PandasUDFType.GROUPED_MAP)
def mean_per_group(pdf: pd.DataFrame) -> pd.DataFrame:
    return pdf.groupby("id").agg({"value": "mean"}).reset_index()

df.groupby("id").apply(mean_per_group)
```

- Input: Pandas DataFrame per group.
- Output: Pandas DataFrame.
- Best for **group-level aggregations**.

### 5. **Iterator Pandas UDF**

```python
@pandas_udf("double")
def scale_batches(iterator):
    for batch in iterator:
        yield batch * 10
```

- Processes data in **batches**.
- Useful for large datasets where memory efficiency matters.

### 6. **Workflow Notes**

- **Enable Arrow:**
    
    ```python
    spark.conf.set("spark.sql.execution.arrow.enabled", "true")
    ```
    
- **Performance Tip:** Always prefer Pandas UDFs over regular UDFs for heavy numeric/string ops.
- **Fallback:** If built-in Spark SQL functions exist, use them first (they’re still fastest).

### Quick Comparison

| Feature         | Regular UDF  | Pandas UDF             | Built-in Function |
| --------------- | ------------ | ---------------------- | ----------------- |
| Execution model | Row-by-row   | Batch (Arrow + Pandas) | Native JVM        |
| Performance     | Slow         | Fast                   | Fastest           |
| Optimizer aware | ❌ No         | ❌ Limited              | ✅ Yes             |
| Best for        | Custom logic | Vectorized ops         | Standard ops      |

## SQL Integration
### 1. Spark SQL Integration

- **`spark.sql()`**: Run SQL queries directly on Spark-managed tables or temporary views.
- Supports **ANSI SQL syntax**, making it familiar for database users.
- Example:
    
    ```python
    spark.sql("SELECT name, age FROM people WHERE age > 20")
    ```
    

### 2. DataFrame API ↔ SQL Queries

- DataFrames can be registered as **temporary views**:
    
    ```python
    df.createOrReplaceTempView("people")
    ```
    
- Then queried with SQL:
    
    ```python
    spark.sql("SELECT COUNT(*) FROM people")
    ```
    
- Results of SQL queries are returned as DataFrames, enabling further transformations with Python.

### 3. Unified Execution Engine

- Both SQL queries and DataFrame operations run on the **same distributed Spark engine**, ensuring performance consistency.
- You can **mix APIs**: run SQL, then apply DataFrame functions (or vice versa).
