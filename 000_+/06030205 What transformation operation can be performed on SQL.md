---
up:
  - "[[000_+/060302 Operations|060302 Operations]]"
down:
prev:
topic: false
question: What transformation operation can be performed on SQL?
---
# What transformation operation can be performed on SQL?


> [!Summary] Summary
> Contents

| **Category**          | **Function / Method**   | **SQL Equivalent**        | **Primary Purpose**                                                   |
| --------------------- | ----------------------- | ------------------------- | --------------------------------------------------------------------- |
| **Column Operations** | `select` / `selectExpr` | `SELECT`                  | Picks specific columns or performs math/logic transformations.        |
|                       | `withColumn`            | `SELECT *, (expr) AS...`  | Adds a new column or replaces an existing one.                        |
|                       | `withColumnRenamed`     | `AS` (Alias)              | Renames a column without changing data.                               |
|                       | `drop`                  | (Omit from `SELECT`)      | Removes columns from the result set.                                  |
| **Filtering**         | `filter` / `where`      | `WHERE`                   | Filters rows based on a boolean condition.                            |
|                       | `distinct`              | `DISTINCT`                | Removes rows that are 100% identical.                                 |
|                       | `dropDuplicates`        | `ROW_NUMBER()` + Filter   | Removes duplicates based on specific subset columns.                  |
| **Aggregation**       | `groupBy` + `agg`       | `GROUP BY`                | Collects data into groups and calculates sums, averages, etc.         |
|                       | `pivot`                 | `PIVOT` / `CASE WHEN`     | Rotates data from rows into columns (cross-tab).                      |
|                       | `rollup` / `cube`       | `ROLLUP` / `CUBE`         | Creates hierarchical or combinatorial grand totals.                   |
| **Joining**           | `join`                  | `JOIN`                    | Combines two datasets based on a common key.                          |
|                       | `crossJoin`             | `CROSS JOIN`              | Creates a Cartesian product (every row with every row).               |
| **Sorting**           | `orderBy` / `sort`      | `ORDER BY`                | Performs a global sort (requires data shuffling).                     |
|                       | `sortWithinPartitions`  | `SORT BY`                 | Sorts data locally within partitions (no shuffle).                    |
| **Set Operations**    | `union` / `unionAll`    | `UNION ALL`               | Merges two datasets (keeps all duplicates).                           |
|                       | `unionByName`           | (Manual alignment)        | Merges datasets by matching column names rather than position.        |
|                       | `intersect` / `except`  | `INTERSECT` / `EXCEPT`    | Finds common rows or rows unique to the first dataset.                |
| **Windowing**         | `Window.partitionBy`    | `OVER (PARTITION BY)`     | Performs calculations across a range of rows without collapsing them. |
|                       | `rank` / `row_number`   | `RANK()` / `ROW_NUMBER()` | Assigns numerical rankings to rows within a window.                   |
|                       | `lead` / `lag`          | `LEAD()` / `LAG()`        | Accesses data from subsequent or previous rows.                       |
| **Missing Data**      | `na.drop`               | `IS NOT NULL`             | Removes rows containing null values.                                  |
|                       | `na.fill`               | `COALESCE`                | Replaces nulls with a default constant value.                         |
|                       | `na.replace`            | `CASE WHEN`               | Swaps specific values (like "N/A") for others.                        |
| **Partitioning**      | `repartition`           | `REPARTITION` hint        | Increases/decreases partitions via a full shuffle.                    |
|                       | `coalesce`              | `COALESCE` hint           | Decreases partitions efficiently without a full shuffle.              |
| **Sampling**          | `sample`                | `TABLESAMPLE`             | Returns a random subset percentage of the data.                       |
|                       | `randomSplit`           | (Manual `RAND()`)         | Splits data into multiple sets (e.g., Train/Test).                    |
| **Streaming**         | `withWatermark`         | (Table Properties)        | Handles late-arriving data in real-time streams.                      |


## Column Operations 

### `select` and `selectExpr`
These are used to pick specific columns or perform transformations.
- **`select`**: Best for standard column selection.
- **`selectExpr`**: A powerful variant that allows you to write SQL-like expressions directly within the DataFrame API.

```sql
SELECT name, age + 1 AS next_year_age FROM users;
```

```python
# select
df.select("name", (df.age + 1).alias("next_year_age"))

# selectExpr (Cleaner for math/logic)
df.selectExpr("name", "age + 1 AS next_year_age")
```

### `withColumn`
This method is used to **add a new column** or **replace an existing one** with the same name. It is the most common way to perform row-wise transformations.

```sql
SELECT *, (salary * 0.1) AS bonus FROM employees;
```

```python
df.withColumn("bonus", df.salary * 0.1)
```

### `withColumnRenamed`
As the name suggests, this is used specifically to rename a column. Unlike `withColumn`, it doesn't transform data; it only changes the metadata (the header).

```sql
SELECT user_id AS id, username FROM users;
```

```python
df.withColumnRenamed("user_id", "id")
```

### `drop`
This removes one or more columns from the DataFrame. In SQL, there is no direct `DROP COLUMN` for a query result; you simply omit the column name from your `SELECT` statement.

```sql
-- You simply don't select the column you don't want
SELECT name, age FROM users; -- (Ignoring 'address' column)
```

```python
# Drop a single column
df.drop("address")

# Drop multiple columns
df.drop("address", "phone_number")
```


## Filtering and Deduplication

### `filter` (and `where`)
In Spark, `filter` and `where` are identical. They allow you to select specific rows based on a condition (Boolean logic).

```sql
SELECT * FROM orders WHERE amount > 100 AND status = 'shipped';
```

```python
# You can use SQL strings or Column objects
df.filter("amount > 100 AND status = 'shipped'")

# Or using Column objects (cleaner for complex logic)
from pyspark.sql.functions import col
df.filter((col("amount") > 100) & (col("status") == "shipped"))
```

### `distinct`

This is used to remove **completely identical** rows. It compares every column in the row to find duplicates.

```sql
SELECT DISTINCT * FROM users;
```

```python
df.distinct()
```

### `dropDuplicates`

This is the "smarter" cousin of `distinct`. While `distinct` looks at the whole row, `dropDuplicates` allows you to specify **subset columns** to define what counts as a duplicate. This is incredibly useful when you have multiple entries for the same ID and only want to keep one (usually the first one Spark encounters).

In standard SQL, you often achieve this using `ROW_NUMBER()` or a `GROUP BY` with aggregate functions, as there isn't a direct "drop duplicates on subset" keyword.

```sql
-- Example: Keeping only the most recent entry per user (conceptual)
SELECT user_id, email, last_login 
FROM (
  SELECT *, ROW_NUMBER() OVER(PARTITION BY user_id ORDER BY last_login DESC) as rank 
  FROM users
) WHERE rank = 1;
```

```python
# Remove rows where the 'user_id' is the same, regardless of other columns
df.dropDuplicates(["user_id"])

# If no columns are provided, it behaves exactly like .distinct()
df.dropDuplicates()
```

## Grouping and Aggregation 

### `groupBy` & `agg`

`groupBy` collects data into groups based on specific columns. It is almost always paired with `agg` (aggregate) to perform calculations like `sum`, `avg`, or `count` on the grouped data.

```sql
SELECT department, SUM(salary) as total_sal, AVG(age) as avg_age 
FROM employees 
GROUP BY department;
```

```python
from pyspark.sql import functions as F

df.groupBy("department").agg(
    F.sum("salary").alias("total_sal"),
    F.avg("age").alias("avg_age")
)
```

### `pivot`

`pivot` rotates data from a state of rows to columns. It’s perfect for creating "cross-tab" reports (e.g., seeing total sales per year, where each year becomes its own column).

```sql
-- Standard SQL uses a CASE WHEN pattern or a specific PIVOT clause
SELECT * FROM (
  SELECT product, quarter, amount FROM sales
) PIVOT (SUM(amount) FOR quarter IN ('Q1', 'Q2', 'Q3', 'Q4'));
```

```python
df.groupBy("product").pivot("quarter").sum("amount")
```

### `rollup`
`rollup` creates hierarchical summaries. If you rollup by `country` and `city`, it will give you:
1. Totals for every **City** within a **Country**.
2. Totals for every **Country** (aggregating all its cities).
3. A **Grand Total** for everything.

```sql
SELECT country, city, SUM(sales) 
FROM table 
GROUP BY ROLLUP (country, city);
```

```python
df.rollup("country", "city").sum("sales")
```


### `cube`

`cube` takes `rollup` a step further by generating totals for **every possible combination** of the columns provided. Using `country` and `city`, it would show:
1. Country + City totals.
2. Country totals (regardless of city).
3. City totals (regardless of country).
4. Grand Total.

```sql
SELECT country, city, SUM(sales) 
FROM table 
GROUP BY CUBE (country, city);
```


```python
df.cube("country", "city").sum("sales")
```

### Joining 

### `join`
A standard join combines two DataFrames based on a common column (the "key"). You must specify the join type (e.g., `inner`, `left`, `right`, `outer`), with `inner` being the default.

```sql
SELECT e.name, d.dept_name
FROM employees e
JOIN departments d ON e.dept_id = d.id;
```

```python
# Standard inner join
df_emp.join(df_dept, df_emp.dept_id == df_dept.id, "inner")

# Using a single string if the column names are identical in both
df_emp.join(df_dept, "dept_id", "left")
```

#### `crossJoin`
A `crossJoin` (or Cartesian Product) matches every single row from the first DataFrame with every single row from the second. If Table A has 10 rows and Table B has 10 rows, the result is **100 rows**.

> **Warning:** Use this sparingly! On large datasets, a cross join can easily crash your Spark cluster by generating an astronomical number of rows.

```sql
SELECT * FROM colors 
CROSS JOIN sizes;
-- Or the old-school way:
SELECT * FROM colors, sizes;
```

```python
df_colors.crossJoin(df_sizes)
```

### Join Types Breakdown

|**Type**|**Result**|
|---|---|
|**inner**|Only rows where keys match in **both** tables.|
|**left**|All rows from left table + matches from right (nulls otherwise).|
|**right**|All rows from right table + matches from left (nulls otherwise).|
|**outer/full**|All rows from both tables, filling nulls where no match exists.|
|**left_semi**|Only rows from left that **have** a match in right (returns left columns only).|
|**left_anti**|Only rows from left that **do NOT have** a match in right.|

## Sorting 


###  `orderBy` (and `sort`)

In Spark, `orderBy` and `sort` are aliases for the same thing. They guarantee a **global sort** of the data. To achieve this, Spark must perform a "shuffle," moving data across the network so that all rows are in the correct order relative to every other row in the entire dataset.

```sql
SELECT * FROM sales ORDER BY transaction_date DESC, amount ASC;
```

```python
from pyspark.sql.functions import col

# Using strings
df.orderBy("transaction_date", ascending=False)

# Using Column objects for mixed directions
df.sort(col("transaction_date").desc(), col("amount").asc())
```

### `sortWithinPartitions`

- This operation sorts data **locally** within each Spark partition but does _not_ move data between them. This means the final DataFrame is not globally sorted, but each individual "chunk" of data is.
- It is significantly faster because it avoids the expensive "shuffle" across the cluster. It’s often used as an optimization step before saving data to certain file formats (like Parquet) to improve compression and filtering performance.

Standard SQL does not have a direct equivalent to "local" sorting, as SQL typically treats the result set as a single logical entity. However, in Hive/Spark SQL, you can use `DISTRIBUTE BY` + `SORT BY`.

```sql
-- This sorts data within each partition
SELECT * FROM sales SORT BY transaction_date;
```

```python
df.sortWithinPartitions("transaction_date")
```

### Key Comparison

|**Feature**|**orderBy / sort**|**sortWithinPartitions**|
|---|---|---|
|**Scope**|Global (entire dataset)|Local (within each partition)|
|**Performance**|Expensive (requires Shuffling)|Fast (no data movement)|
|**Guarantee**|Row 1 < Row 2 ... Row N|Row 1 < Row 2 (only within partition A)|
|**Use Case**|Final reporting and top-N analysis|Performance optimization for writes|

## Set Operations 


###  `union` & `unionAll`

In Spark, `union` and `unionAll` are **identical**. Both combine two DataFrames and **keep all rows**, including duplicates. This is a common point of confusion because, in standard SQL, `UNION` removes duplicates while `UNION ALL` keeps them. In Spark, if you want to remove duplicates after a union, you must explicitly call `.distinct()`.

```sql
SELECT * FROM table1
UNION ALL
SELECT * FROM table2;
```

```python
df1.union(df2)
# Or
df1.unionAll(df2)
```

### `unionByName`
Standard `union` merges data based on **column position** (Column 1 of DF A matches Column 1 of DF B). `unionByName` merges based on **column names**. This is much safer if your DataFrames have the same columns but in a different order.

SQL doesn't have a direct `UNION BY NAME`. You have to manually align columns in your `SELECT` statements.

```python
# If df1 is (name, age) and df2 is (age, name)
df1.unionByName(df2)
```

### `intersect` & `intersectAll`
These find the rows that exist in **both** DataFrames.
- **`intersect`**: Returns unique rows found in both (removes duplicates).
- **`intersectAll`**: Returns rows found in both, preserving the duplicate count.
    
```sql
SELECT * FROM table1 INTERSECT SELECT * FROM table2;
```

```python
df1.intersect(df2)
df1.intersectAll(df2)
```

### `except` & `exceptAll`
These return rows that exist in the first DataFrame but **not** in the second. (In some SQL dialects, this is known as `MINUS`).
- **`except`**: Returns unique rows from the left DF not in the right DF.
- **`exceptAll`**: Returns rows from the left DF not in the right DF, preserving duplicate counts.

```sql
SELECT * FROM table1 EXCEPT SELECT * FROM table2;
```

```python
df1.exceptAll(df2) # or df1.subtract(df2)
```

### Windowing 

In Spark, **Window functions** allow you to perform calculations across a "window" of rows related to the current row. Unlike `groupBy`, which collapses rows into a single summary, Window functions keep the individual rows intact while adding an aggregated value (like a running total or a rank) to each one.
To use Window functions, you generally need three things:
1. **Partitioning**: Dividing data into groups (e.g., by `department`).
2. **Ordering**: Defining the sequence within those groups (e.g., by `salary`).
3. **Frame**: (Optional) Defining the boundaries relative to the current row (e.g., "last 3 rows").

#### Defining the Window
In the DataFrame API, you must first define a `WindowSpec`.

```sql
-- Calculating a rank and a running total
SELECT 
    name, 
    dept, 
    salary,
    RANK() OVER (PARTITION BY dept ORDER BY salary DESC) as rank,
    SUM(salary) OVER (PARTITION BY dept ORDER BY salary ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) as running_total
FROM employees;
```


```python
from pyspark.sql.window import Window
from pyspark.sql import functions as F

# 1. Define the window
windowSpec = Window.partitionBy("dept").orderBy(F.col("salary").desc())

# 2. Apply the functions
df.withColumn("rank", F.rank().over(windowSpec)) \
  .withColumn("running_total", F.sum("salary").over(windowSpec.rowsBetween(Window.unboundedPreceding, Window.currentRow)))
```

#### Ranking Functions
- **`row_number()`**: Unique sequential number (1, 2, 3, 4).
- **`rank()`**: Handles ties by skipping numbers (1, 2, 2, 4).
- **`dense_rank()`**: Handles ties without skipping (1, 2, 2, 3).

#### Analytic Functions
- **`lead(col, n)`**: Accesses data from the row **$n$** positions ahead.
- **`lag(col, n)`**: Accesses data from the row **$n$** positions behind.
    
#### Aggregate Functions

- **`sum()`, `avg()`, `min()`, `max()`**: Calculated over the defined window frame.

#### Window Frames

The "Frame" determines exactly which rows are included in the calculation relative to the current row.

|**Frame Type**|**Meaning**|
|---|---|
|**`unboundedPreceding`**|From the very start of the partition.|
|**`unboundedFollowing`**|To the very end of the partition.|
|**`currentRow`**|The row currently being processed.|
|**`rowsBetween(-1, 1)`**|One row before, the current row, and one row after.|


## Handling Missing Data

### `na.drop`

This is used to remove rows that contain null values. You can control how aggressive the dropping is by using the `how` and `subset` parameters.

```sql
-- Remove rows where ANY column is NULL
SELECT * FROM table WHERE col1 IS NOT NULL AND col2 IS NOT NULL...;

-- Specific column check
SELECT * FROM table WHERE name IS NOT NULL;
```

```python
# Drop row if ANY column has a null
df.na.drop() 

# Drop row only if ALL columns are null
df.na.drop(how="all")

# Drop row if nulls exist in specific columns
df.na.drop(subset=["email", "phone"])
```


### `na.fill`

This replaces null values with a specific constant. Spark is smart enough to match types—if you provide a string, it will only fill nulls in string columns.

```sql
-- Using COALESCE is the standard way to fill nulls
SELECT COALESCE(description, 'No description provided') AS desc FROM products;
```

```python
# Replace all nulls in numeric columns with 0
df.na.fill(0)

# Replace nulls in specific columns with specific values
df.na.fill({"name": "Unknown", "age": 18})
```

### `na.replace`

While `na.fill` specifically targets `null`, `na.replace` is more flexible. It allows you to swap out specific values (like "N/A", "Unknown", or -1) for something else, or replace nulls by targeting them explicitly in a map.

```sql
SELECT CASE 
    WHEN status = 'Old' THEN 'Legacy' 
    WHEN status = 'TBD' THEN 'Pending' 
    ELSE status 
END AS status FROM orders;
```

```python
# Replace 'Old' with 'Legacy' and 'TBD' with 'Pending' in the 'status' column
df.na.replace({"Old": "Legacy", "TBD": "Pending"}, subset=["status"])
```

### Comparison of Null Handling

|**Operation**|**Target**|**Best For...**|
|---|---|---|
|**`na.drop`**|The entire row|Cleaning datasets where incomplete records are useless.|
|**`na.fill`**|Null values only|Providing defaults for missing data (e.g., 0 for counts).|
|**`na.replace`**|Specific values|Standardizing messy data (e.g., mapping "Unknown" to null).|


## Partitioning

### `repartition`

This operation increases or decreases the number of partitions. It performs a **full shuffle**, meaning data is scrambled across the network to ensure that every new partition is roughly the same size.

In Spark SQL, you use the `REPARTITION` hint.

```sql
SELECT /*+ REPARTITION(10) */ * FROM large_table;

-- You can also repartition by specific columns
SELECT /*+ REPARTITION(5, department) */ * FROM employees;
```

```python
# Increase or decrease to exactly 10 partitions (Round Robin)
df.repartition(10)

# Partition based on a column (useful for joins/grouping)
df.repartition(5, "country")
```

### `repartitionByRange`

Unlike standard repartitioning which uses a hash (effectively random), `repartitionByRange` uses the data's **actual values** to create partitions. It ensures that rows with similar values end up in the same partition and that those partitions are ordered.

> **Why use this?** It is excellent for data that has a natural order (like dates or IDs), as it makes range-based queries and sorting significantly faster later on.

```sql
SELECT /*+ REPARTITION_BY_RANGE(5, transaction_date) */ * FROM sales;
```

```python
# Partitions will be sorted: Partition 1 (Jan), Partition 2 (Feb), etc.
df.repartitionByRange(5, "transaction_date")
```

### `coalesce`

This is an optimized version of `repartition` used **only to decrease** the number of partitions. Because it doesn't try to create a perfect balance, it avoids a full shuffle. It simply merges existing partitions.

```sql
SELECT /*+ COALESCE(2) */ * FROM table;
```

```python
# Efficiently reduces partitions from 1000 down to 10
df.coalesce(10)
```

### The Golden Rule of Partitions
- **Too many partitions:** High overhead (Spark spends more time managing tasks than processing data).
- **Too few partitions:** "Out of Memory" errors and under-utilized CPUs.
- **The Sweet Spot:** Usually aim for partitions that are roughly **128MB to 200MB** in size when stored in memory.

## Sampling

### `sample`

This method returns a random subset of the original DataFrame based on a fraction. It is typically used for **exploration or visualization** when the full dataset is too large to handle.

#### Key Parameters:
- **withReplacement**: If `True`, a row can be sampled multiple times.
- **fraction**: The percentage of data to return (e.g., `0.1` for 10%).
- **seed**: A fixed number to ensure you get the same "random" result every time you run the code.

```sql
-- Returns a 10% sample of the data
SELECT * FROM users TABLESAMPLE (10 PERCENT);
```

```python
# Sample 10% of the data without replacement
df_sample = df.sample(withReplacement=False, fraction=0.1, seed=42)
```

### `randomSplit`
This method splits a single DataFrame into **multiple** DataFrames. It is the standard tool for creating **Training, Validation, and Test sets** in Machine Learning.

Unlike `sample`, the weights you provide should ideally sum up to 1 (e.g., `[0.8, 0.2]`), and Spark ensures that the rows are distributed across the resulting DataFrames so that there is no overlap.

Standard SQL does not have a native `randomSplit` function. You would typically achieve this using `RAND()` and `WHERE` clauses, which is much more manual.

```python
# Split data into 80% training and 20% testing
train_df, test_df = df.randomSplit([0.8, 0.2], seed=42)

print(f"Train count: {train_df.count()}")
print(f"Test count: {test_df.count()}")
```

### A Small Warning on Reproducibility

If you use these functions on a cluster and your underlying data changes or the number of partitions changes, the "random" selection might change even if you use the same `seed`. If you need 100% consistency across different runs on changing data, it's often safer to:
1. Add a unique ID or hash to your rows.
2. Filter based on that ID (e.g., `WHERE id % 10 == 0` for a 10% sample).
    
## Streaming 

In the world of Spark Structured Streaming, `withWatermark` is a crucial concept used to handle **late data**.

When processing real-time streams, data doesn't always arrive in the order it was generated. A sensor might go offline and upload its data 10 minutes late. Without a watermark, Spark would have to keep all old state in memory forever "just in case" more data arrives. A watermark tells Spark: _"Stop waiting for data older than X amount of time."_

### 1. How it works
You define a column containing the event time (timestamp) and a threshold (delay).
- **Event Time**: When the event actually happened.
- **Threshold**: How long you are willing to wait for late arrivals (e.g., "10 minutes").

Spark tracks the maximum event time seen so far. If the max time is 12:00 PM and your watermark is 10 minutes, Spark will drop any data with a timestamp older than 11:50 AM.

### DataFrame (Streaming)

```python
from pyspark.sql.functions import window, col

# Assume 'event_time' is our timestamp column
streaming_df = spark.readStream ... 

# Set a watermark of 10 minutes
windowed_counts = streaming_df \
    .withWatermark("event_time", "10 minutes") \
    .groupBy(
        window(col("event_time"), "5 minutes"), # 5-minute windows
        col("sensor_id")
    ).count()
```

### SQL (Streaming)

In Spark SQL, the watermark is often defined in the `JOIN` or `GROUP BY` logic if using a streaming provider:

```python
-- Conceptual SQL for streaming
SELECT 
    window.start, 
    window.end, 
    sensor_id, 
    COUNT(*)
FROM streaming_table
-- Watermarking is often set via table properties or specific hints
GROUP BY window(event_time, '5 minutes'), sensor_id;
```

### Why is it necessary?
1. **Memory Management**: It allows Spark to clear "state" (old aggregates) from memory. Once the watermark passes a window's end time, Spark emits the final result and deletes that window from its RAM.
2. **Accuracy**: It ensures that late-arriving data is still included in the correct aggregate bucket, provided it arrives within the allowed delay.
3. **Join Support**: It is mandatory for **Stream-Stream Joins** to prevent the state from growing infinitely.

### Key Rules for Watermarking

|**Rule**|**Description**|
|---|---|
|**Output Mode**|Must be `Append` or `Update`. Watermarking does not work with `Complete` mode (since `Complete` keeps all data anyway).|
|**Placement**|You must call `withWatermark` **before** the aggregation or join.|
|**Column Type**|The column must be of `TimestampType`.|
