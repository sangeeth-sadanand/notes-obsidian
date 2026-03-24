
## Categories of Operators in PySpark DataFrames

### **Arithmetic Operators**

Used for mathematical transformations on numeric columns.

- `+` → Addition
- `-` → Subtraction
- `*` → Multiplication
- `/` → Division
- `%` → Modulo
- `abs()` → Absolute value
- `pow()` → Power/exponentiation

```python
df.select((col("salary") * 1.1).alias("updated_salary")).show()
```

### **Comparison Operators**

Used for filtering or conditional logic.

- `==` or `eqNullSafe()` → Equality check
- `!=` → Not equal
- `>` , `<` , `>=` , `<=` → Greater/less comparisons
- `between()` → Range check

```python
df.filter(col("age") > 30).show()
```

### **Logical Operators**

Combine multiple conditions.

- `&` → AND
- `|` → OR
- `~` → NOT

```python
df.filter((col("age") > 30) & (col("salary") > 50000)).show()
```
### **String Pattern Operators**

Work with text columns.

- `like()` → SQL LIKE pattern
- `rlike()` → Regex pattern match
- `ilike()` → Case-insensitive LIKE
- `startswith()`, `endswith()` → Prefix/suffix checks

```python
df.filter(col("name").rlike("^A")).show()
```

### **Null Handling Operators**

Special operators for dealing with missing values.

- `isNull()` / `isNotNull()`
- `coalesce()` → First non-null value
- `when()` / `otherwise()` → Conditional expressions
- `nanvl()` → Replace NaN with a value

```python
df.select(coalesce(col("middle_name"), lit("N/A"))).show()
```

## Functions

### 1. **Column & Expression Functions**

- `col` → reference columns
- `lit` → add constants
- `expr` → SQL-like expressions
- `selectExpr` → multiple SQL expressions
- `alias` → rename columns
- `cast` → change column type

```python
# Examples
result = (
    df
    # col: reference column
    .select(
        col("name"),
        col("age") + 5,  # add 5 to age
        # lit: add constant value
        lit("India").alias("country"),
        # expr: SQL-like expression
        expr("salary + 500").alias("bonus_salary"),
        # alias: rename column
        col("salary").alias("monthly_salary"),
        # cast: change type
        col("age").cast("string").alias("age_str")
    )
)
# selectExpr: multiple SQL-style expressions at once
result2 = df.selectExpr(
    "name",
    "age + 10 as age_in_10yrs",
    "salary * 1.1 as updated_salary"
)
```

### 2. **Math Functions**
- `abs` → absolute value
- `round` → rounds  to nearest integer (or decimal places if specified)
- `sqrt` → square root 
- `pow` → raises  to a power 
- `exp` → exponential 
- `log` → natural log 
- `rand` → generates random number between 0 and 1
- `floor` → rounds down
- `ceil` → rounds up


```python

result = (
    df
    .select(
        col("name"),
        # abs: absolute value
        abs(col("age")).alias("abs_age"),
        # round: round to nearest integer
        round(col("salary"), 0).alias("rounded_salary"),
        # sqrt: square root
        sqrt(col("salary")).alias("sqrt_salary"),
        # pow: power function
        pow(col("age"), 2).alias("age_squared"),
        # exp: exponential
        exp(col("age")).alias("exp_age"),
        # log: natural logarithm
        log(col("salary")).alias("log_salary"),
        # rand: random number between 0 and 1
        rand().alias("random_val"),
        # floor: round down
        floor(col("salary")).alias("floor_salary"),
        # ceil: round up
        ceil(col("salary")).alias("ceil_salary")
    )
)

result.show(truncate=False)
```

### 3. **String Functions**

- `concat` → joins columns directly
- `concat_ws` → joins with separator
- `substring` → extract substring
- `length` → string length
- `lower` / `upper` → case conversion
- `trim` → remove leading/trailing spaces
- `lpad` / `rpad` → pad string left/right
- `regexp_replace` → replace regex pattern
- `regexp_extract` → extract regex group
- `like` → SQL LIKE pattern match
- `rlike` → regex match
- `startswith` / `endswith` → prefix/suffix check

```python
# Apply string functions
result = (
    df
    .select(
        # concat: join columns directly
        concat(col("name"), col("country")).alias("concat_name_country"),
        # concat_ws: join with separator
        concat_ws("-", col("name"), col("country")).alias("concat_ws"),
        # substring: extract substring (start=1, length=3)
        substring(col("name"), 1, 3).alias("substring_name"),
        # length: string length
        length(col("name")).alias("name_length"),
        # lower: lowercase
        lower(col("country")).alias("country_lower"),
        # upper: uppercase
        upper(col("country")).alias("country_upper"),
        # trim: remove spaces
        trim(col("name")).alias("trimmed_name"),
        # lpad: pad left with '*'
        lpad(col("name"), 10, "*").alias("lpad_name"),
        # rpad: pad right with '*'
        rpad(col("name"), 10, "*").alias("rpad_name"),
        # regexp_replace: replace pattern
        regexp_replace(col("country"), "U", "X").alias("regexp_replace_country"),
        # regexp_extract: extract regex group
        regexp_extract(col("country"), "(I.*)", 1).alias("regexp_extract_country"),
        # like: SQL LIKE
        col("country").like("I%").alias("like_country"),
        # rlike: regex match
        col("country").rlike("^U.*").alias("rlike_country"),
        # startswith
        col("country").startswith("U").alias("startswith_country"),
        # endswith
        col("country").endswith("A").alias("endswith_country")
    )
)

```

### 4. **Date & Time Functions**
- `current_date` → today’s date
- `current_timestamp` → current date + time
- `year`, `month`, `dayofmonth` → extract parts of a date
- `hour`, `minute`, `second` → extract parts of a timestamp
- `datediff` → difference in days between two dates
- `date_add` / `date_sub` → add or subtract days
- `add_months` → shift by months
- `last_day` → last day of the month for a given date

```python
result = (
    df
    .select(
        col("name"),
        col("join_date"),
        # current_date and current_timestamp
        current_date().alias("current_date"),
        current_timestamp().alias("current_timestamp"),
        # extract parts
        year(col("join_date")).alias("year"),
        month(col("join_date")).alias("month"),
        dayofmonth(col("join_date")).alias("day"),
        hour(current_timestamp()).alias("hour"),
        minute(current_timestamp()).alias("minute"),
        second(current_timestamp()).alias("second"),
        # date arithmetic
        datediff(current_date(), col("join_date")).alias("days_since_join"),
        date_add(col("join_date"), 10).alias("date_plus_10"),
        date_sub(col("join_date"), 5).alias("date_minus_5"),
        add_months(col("join_date"), 2).alias("plus_2_months"),
        last_day(col("join_date")).alias("last_day_of_month")
    )
)
```


### 5. **Conditional Functions**

- `when` / `otherwise` → conditional logic (like SQL CASE)
- `coalesce` → returns first non-null value among columns
- `isnull` / `isnotnull` → check for null values
- `greatest` → maximum across multiple columns
- `least` → minimum across multiple columns

```python
from pyspark.sql.functions import (
    col, when, coalesce,
    greatest, least
)
result = (
    df
    .select(
        col("name"),
        col("age1"),
        col("age2"),
        col("salary"),
        # when + otherwise: conditional logic
        when(col("age1").isNull(), 0)
            .otherwise(col("age1"))
            .alias("age1_filled"),
        # coalesce: first non-null value
        coalesce(col("age1"), col("age2")).alias("coalesce_age"),
        # isnull / isnotnull
        col("salary").isNull().alias("salary_is_null"),
        col("salary").isNotNull().alias("salary_is_not_null"),
        # greatest: max of columns
        greatest(col("age1"), col("age2")).alias("max_age"),
        # least: min of columns
        least(col("age1"), col("age2")).alias("min_age")
    )
)
```


### 6. **Aggregate Functions**

- **count(*)** → Counts all rows in the DataFrame.
- **count("col")** → Counts non‑null values in a column.
- **countDistinct("col")** → Counts unique values in a column.
- **sum("col")** → Adds up all numeric values in a column.
- **avg("col") / mean("col")** → Computes the average of numeric values.
- **max("col")** → Finds the largest value in a column.
- **min("col")** → Finds the smallest value in a column.
- **first("col")** → Returns the first value encountered in a column.
- **last("col")** → Returns the last value encountered in a column.
- **collect_list("col")** → Aggregates values into a list (duplicates kept).
- **collect_set("col")** → Aggregates values into a set (duplicates removed).
- **var_pop("col")** → Population variance of numeric values.
- **var_samp("col")** → Sample variance of numeric values.
- **stddev_pop("col")** → Population standard deviation of numeric values.
- **stddev_samp("col")** → Sample standard deviation of numeric values.

```python
from pyspark.sql import functions as F

# Apply aggregate functions
agg_df = df.agg(
    F.count("*").alias("count_all"),
    F.count("age").alias("count_age"),
    F.countDistinct("age").alias("count_distinct_age"),
    F.sum("salary").alias("sum_salary"),
    F.avg("salary").alias("avg_salary"),
    F.mean("salary").alias("mean_salary"),  # same as avg
    F.max("salary").alias("max_salary"),
    F.min("salary").alias("min_salary"),
    F.first("name").alias("first_name"),
    F.last("name").alias("last_name"),
    F.collect_list("name").alias("collect_list_names"),
    F.collect_set("age").alias("collect_set_ages"),
    F.var_pop("salary").alias("var_pop_salary"),
    F.var_samp("salary").alias("var_samp_salary"),
    F.stddev_pop("salary").alias("stddev_pop_salary"),
    F.stddev_samp("salary").alias("stddev_samp_salary")
)

```


### 7. **Array & Map Functions**

- **array()** → Creates an array column from multiple input columns.
- **size(array_col)** → Returns the number of elements in an array.
- **explode(array_col)** → Flattens an array into multiple rows (one per element).
- **array_contains(array_col, value)** → Checks if an array contains a specific value.
- **map(key, value, …)** → Creates a map column from key‑value pairs.
- **map_keys(map_col)** → Extracts all keys from a map as an array.
- **map_values(map_col)** → Extracts all values from a map as an array.

```python
# Array functions
array_df = df.select(
    F.array("age", "salary").alias("age_salary_array"),
    F.size(F.array("age", "salary")).alias("array_size"),
    F.array_contains(F.array("age", "salary"), 34).alias("contains_34"),
    F.explode(F.array("age", "salary")).alias("exploded_values")
)

# Map functions
map_df = df.select(
    F.map(F.lit("age"), "age", F.lit("salary"), "salary").alias("map_col"),
    F.map_keys(F.map(F.lit("age"), "age", F.lit("salary"), "salary")).alias("map_keys"),
    F.map_values(F.map(F.lit("age"), "age", F.lit("salary"), "salary")).alias("map_values")
)
```

### 8. **JSON Functions**

- **get_json_object(json_col, path)** → Extracts a specific field from a JSON string using a JSONPath expression.
- **from_json(json_col, schema)** → Parses a JSON string column into a structured column (array, struct, or map) based on a schema.
- **to_json(struct_col)** → Converts a struct, array, or map column back into a JSON string.

```python
# Apply JSON functions
json_df = df.select(
    F.get_json_object("json_str", "$.name").alias("extracted_name"),
    F.from_json("json_str", schema).alias("parsed_json"),
    F.to_json(F.struct(F.lit("Charlie").alias("name"), F.lit(29).alias("age"))).alias("json_output")
)

json_df.show(truncate=False)
```
### 9. **Window Functions**

- **row_number()** → Assigns a unique sequential number to each row within a partition (no ties).
- **rank()** → Assigns ranks with gaps if there are ties (e.g., 1, 2, 2, 4).
- **dense_rank()** → Assigns ranks without gaps for ties (e.g., 1, 2, 2, 3).
- **lead(col, offset)** → Returns the value of a column from a later row (look ahead).
- **lag(col, offset)** → Returns the value of a column from an earlier row (look back).
- **ntile(n)** → Divides rows into _n_ buckets and assigns a bucket number to each row.

```python
# Define window partitioned by dept, ordered by salary descending
windowSpec = Window.partitionBy("dept").orderBy(F.desc("salary"))

# Apply window functions
result = df.select(
    "name", "dept", "salary",
    F.row_number().over(windowSpec).alias("row_number"),
    F.rank().over(windowSpec).alias("rank"),
    F.dense_rank().over(windowSpec).alias("dense_rank"),
    F.lead("salary", 1).over(windowSpec).alias("lead_salary"),
    F.lag("salary", 1).over(windowSpec).alias("lag_salary"),
    F.ntile(2).over(windowSpec).alias("ntile_2")  # split into 2 buckets
)

```

## Methods

### 1. **Creation & Metadata**

- `columns` → List of column names
- `dtypes` → Column names with data types
- `printSchema()` → Display schema
- `schema` → Schema object

```python
# 1. List of column names
print("Columns:", df.columns)

# 2. Column names with data types
print("Dtypes:", df.dtypes)

# 3. Display schema in tree format
print("Schema via printSchema():")
df.printSchema()

# 4. Schema object
print("Schema object:", df.schema)
```

### 2. **Column Operations**

- `select()`, `selectExpr()` → Choose columns or SQL expressions
- `withColumn()`, `withColumnRenamed()` → Add/rename columns
- `drop()` → Remove columns
- `colRegex()` → Select columns by regex
```python
# 1. select() → choose specific columns
df_select = df.select("Name", "Age")
print("select():")
df_select.show()

# 2. selectExpr() → SQL-like expressions
df_expr = df.selectExpr("Name", "Age + 5 as AgePlusFive")
print("selectExpr():")
df_expr.show()

# 3. withColumn() → add new column
df_newcol = df.withColumn("AgeDouble", col("Age") * 2)
print("withColumn():")
df_newcol.show()

# 4. withColumnRenamed() → rename column
df_renamed = df.withColumnRenamed("Department", "Dept")
print("withColumnRenamed():")
df_renamed.show()

# 5. drop() → remove column
df_dropped = df.drop("Age")
print("drop():")
df_dropped.show()

# 6. colRegex() → select columns by regex
df_regex = df.select(df.colRegex("`.*e.*`"))  # columns containing 'e'
print("colRegex():")
df_regex.show()

```

### 3. **Filtering & Conditional**

- `filter()`, `where()` → Row filtering
- `distinct()` → Unique rows
- `dropDuplicates()` → Remove duplicates
```python

# 1. filter() → filter rows with Age > 30
df_filter = df.filter(col("Age") > 30)
print("filter():")
df_filter.show()

df_where = df.where("Department = 'Finance'")
print("where():")
df_where.show()

# 3. distinct() → unique rows across all columns
df_distinct = df.distinct()
print("distinct():")
df_distinct.show()

# 4. dropDuplicates() → remove duplicates based on specific columns
df_dropdup = df.dropDuplicates(["Age", "Department"])
print("dropDuplicates():")
df_dropdup.show()

```

### 4. **Aggregation & Grouping**

- `groupBy()`, `agg()` → Group and aggregate
- `pivot()` → Pivot tables

```python

df = spark.createDataFrame(data, columns)

# 1. groupBy() + agg() → standard grouping
df_group = df.groupBy("Department").agg(avg("Salary").alias("AvgSalary"))
print("groupBy + agg:")
df_group.show()

# 4. pivot() → pivot table (reshape data)
df_pivot = df.groupBy("Department").pivot("Name").agg(sum("Salary"))
print("pivot:")
df_pivot.show()

```

### 5. **Joins**

- `join()` → Combine DataFrames on keys

```python
# INNER JOIN → only matching rows
inner_join = df_emp.join(df_dept, df_emp.Department == df_dept.DeptCode, "inner")
print("INNER JOIN:")
inner_join.show()

# LEFT JOIN → all rows from left, matches from right
left_join = df_emp.join(df_dept, df_emp.Department == df_dept.DeptCode, "left")
print("LEFT JOIN:")
left_join.show()

# RIGHT JOIN → all rows from right, matches from left
right_join = df_emp.join(df_dept, df_emp.Department == df_dept.DeptCode, "right")
print("RIGHT JOIN:")
right_join.show()

# FULL OUTER JOIN → all rows from both sides
outer_join = df_emp.join(df_dept, df_emp.Department == df_dept.DeptCode, "outer")
print("FULL OUTER JOIN:")
outer_join.show()

# LEFT SEMI JOIN → only rows from left that have a match
semi_join = df_emp.join(df_dept, df_emp.Department == df_dept.DeptCode, "left_semi")
print("LEFT SEMI JOIN:")
semi_join.show()

# LEFT ANTI JOIN → only rows from left that do NOT have a match
anti_join = df_emp.join(df_dept, df_emp.Department == df_dept.DeptCode, "left_anti")
print("LEFT ANTI JOIN:")
anti_join.show()
```

|Join Type|What it Returns|
|---|---|
|**Inner**|Rows with matching keys in both DataFrames|
|**Left**|All rows from left + matches from right|
|**Right**|All rows from right + matches from left|
|**Outer (Full)**|All rows from both sides, with nulls where no match|
|**Left Semi**|Only rows from left that have a match in right (no right-side columns returned)|
|**Left Anti**|Only rows from left that do _not_ have a match in right|

### 6. **Sorting & Sampling**

- `orderBy()`, `sort()` → Sort rows
- `sample()` → Random sampling
- `randomSplit()` → Split into multiple DataFrames
- 
```python
# 1. orderBy() → sort rows (ascending by Age)
df_order = df.orderBy("Age")
print("orderBy():")
df_order.show()

# 2. sort() → same as orderBy, can specify descending
df_sort = df.sort(col("Age").desc())
print("sort():")
df_sort.show()

# 3. sample() → random sample (e.g., 40% of rows, without replacement)
df_sample = df.sample(withReplacement=False, fraction=0.4, seed=42)
print("sample():")
df_sample.show()

# 4. randomSplit() → split DataFrame into multiple parts
splits = df.randomSplit([0.6, 0.4], seed=42)
train_df, test_df = splits[0], splits[1]

print("randomSplit - Train set:")
train_df.show()

print("randomSplit - Test set:")
test_df.show()

```

### 7. **Persistence & Optimization**

- `cache()`, `persist()` → Store DataFrame in memory/disk
- `unpersist()` → Remove cached data
- `checkpoint()` → Save state for fault tolerance
- `coalesce()`, `repartition()` → Control partitions
```python
# 1. cache() → store DataFrame in memory
df_cached = df.cache()
print("cache():")
df_cached.show()

# 2. persist() → store DataFrame with custom storage level (e.g., MEMORY_AND_DISK)
from pyspark import StorageLevel
df_persisted = df.persist(StorageLevel.MEMORY_AND_DISK)
print("persist():")
df_persisted.show()

# 3. unpersist() → remove cached/persisted data
df_persisted.unpersist()

# 4. checkpoint() → save state for fault tolerance (requires checkpoint dir)
df_checkpoint = df.checkpoint()
print("checkpoint():")
df_checkpoint.show()

# 5. coalesce() → reduce number of partitions (no shuffle)
df_coalesce = df.coalesce(1)
print("coalesce() → partitions:", df_coalesce.rdd.getNumPartitions())

# 6. repartition() → increase or redistribute partitions (with shuffle)
df_repartition = df.repartition(4)
print("repartition() → partitions:", df_repartition.rdd.getNumPartitions())
```

### 8. **Actions**

- `show()` → Display rows
- `collect()` → Return all rows as Python objects
- `count()` → Number of rows
- `first()`, `head()`, `take()` → Retrieve rows
```python
# 1. show() → display rows in tabular format
print("show():")
df.show(3)  # show first 3 rows

# 2. collect() → return all rows as list of Row objects
print("collect():")
rows = df.collect()
print(rows)

# 3. count() → number of rows
print("count():", df.count())

# 4. first() → first row
print("first():", df.first())

# 5. head() → first n rows
print("head(2):", df.head(2))

# 6. take() → retrieve n rows (similar to head)
print("take(2):", df.take(2))
```
### 9. **Statistics**

- `describe()` → Summary statistics
- `summary()` → Extended statistics
- `corr()`, `cov()` → Correlation/covariance
- `approxQuantile()` → Approximate quantiles
```python
# 1. describe() → basic summary statistics (count, mean, stddev, min, max)
print("describe():")
df.describe(["Age", "Salary"]).show()

# 2. summary() → extended statistics (adds percentiles, median, etc.)
print("summary():")
df.summary("count", "mean", "min", "25%", "50%", "75%", "max").show()

# 3. corr() → correlation between two numeric columns
print("corr(Age, Salary):", df.stat.corr("Age", "Salary"))

# 4. cov() → covariance between two numeric columns
print("cov(Age, Salary):", df.stat.cov("Age", "Salary"))

# 5. approxQuantile() → approximate quantiles for a column
quantiles = df.stat.approxQuantile("Salary", [0.25, 0.5, 0.75], 0.01)
print("approxQuantile(Salary):", quantiles)
```
### 10. **I/O Operations**

- `write` → Save DataFrame (Parquet, CSV, JSON, etc.)
- `read` (via `spark.read`) → Load DataFrame
```python
# --- WRITE examples ---
# Save as Parquet
df.write.mode("overwrite").parquet("/tmp/employees_parquet")

# Save as CSV (with header)
df.write.mode("overwrite").option("header", True).csv("/tmp/employees_csv")

# Save as JSON
df.write.mode("overwrite").json("/tmp/employees_json")

# --- READ examples ---
# Read Parquet
df_parquet = spark.read.parquet("/tmp/employees_parquet")
print("Read Parquet:")
df_parquet.show()

# Read CSV
df_csv = spark.read.option("header", True).csv("/tmp/employees_csv")
print("Read CSV:")
df_csv.show()

# Read JSON
df_json = spark.read.json("/tmp/employees_json")
print("Read JSON:")
df_json.show()
```
## Window methods


- `__partitionBy(cols)` → Defines how rows are grouped into partitions (like SQL `PARTITION BY`).
- `orderBy(cols)` → Defines ordering of rows within each partition (like SQL `ORDER BY`).
- `rowsBetween(start, end)` → Defines a frame based on physical row offsets relative to the current row.
- `rangeBetween(start, end)` → Defines a frame based on value ranges relative to the current row.

### Frame Bound Constants

Used with `rowsBetween` and `rangeBetween`:

- **Window.unboundedPreceding** → Start from the first row in the partition.
- **Window.unboundedFollowing** → End at the last row in the partition.
- **Window.currentRow** → Refers to the current row.

```python
# Define window with partition, order, and frame
windowSpec = (
    Window.partitionBy("dept")
          .orderBy(F.desc("salary"))
          .rowsBetween(Window.unboundedPreceding, Window.currentRow)
)

result = df.select(
    "name", "dept", "salary",
    F.sum("salary").over(windowSpec).alias("running_sum")
)
```
