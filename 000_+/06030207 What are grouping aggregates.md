---
up:
  - "[[000_+/060302 Operations|060302 Operations]]"
down:
prev:
topic: false
question: What are grouping aggregates?
---
# What are grouping aggregates?


> [!Summary] Summary
> Contents

- In PySpark, **grouping aggregates** are the bread and butter of data analysis. 
- They allow you to collapse a large dataset into meaningful summaries by "grouping" rows that share the same value in one or more columns and then calculating a metric (like a sum or average) for each group.

The underlying logic follows the **Split-Apply-Combine** strategy:
1. **Split:** The data is partitioned into groups based on specific keys.
2. **Apply:** An aggregate function is calculated for each group.
3. **Combine:** The results are merged into a single output DataFrame.

## The Core Syntax
In PySpark, grouping is almost always a two-step process:
1. Call `.groupBy("column_name")`
2. Follow it immediately with an aggregate function like `.sum()`, `.avg()`, `.count()`, `.min()`, or `.max()`.

If you need to perform multiple different calculations at once, you use the `.agg()` function.

## Practical Code Example
Let's look at a scenario involving sales data. We want to see how different departments are performing.

```python
from pyspark.sql import SparkSession
from pyspark.sql import functions as F

# Initialize Spark Session
spark = SparkSession.builder.appName("GroupingAggregates").getOrCreate()

# Sample Data: Department, Employee, and Sales Amount
data = [
    ("Sales", "Alice", 5000),
    ("Sales", "Bob", 7000),
    ("IT", "Charlie", 3000),
    ("IT", "David", 4000),
    ("IT", "Eve", 2000),
    ("HR", "Frank", 2500),
    ("HR", "Grace", 2500)
]

columns = ["Department", "Employee", "Salary"]
df = spark.createDataFrame(data, schema=columns)

# 1. Simple Grouping: Total Salary per Department
df_sum = df.groupBy("Department").sum("Salary")
df_sum.show()

# 2. Grouping by Multiple Columns
# (In this case, it returns unique pairs since names are unique)
df_multi = df.groupBy("Department", "Employee").count()
df_multi.show()
```

## Using the `.agg()` Function
While `.sum()` and `.avg()` are convenient, they are limited. If you want to calculate the **Average**, **Total**, and **Max** salary for each department all in one go, you use `.agg()`.

This is the "pro" way to do it because it allows you to alias your columns immediately, keeping your code clean.

```python
# Multiple aggregates using F.functions
df_stats = df.groupBy("Department").agg(
    F.sum("Salary").alias("Total_Payroll"),
    F.avg("Salary").alias("Average_Salary"),
    F.max("Salary").alias("Highest_Salary"),
    F.count("Employee").alias("Headcount")
)

df_stats.show()
```

## Key Aggregate Functions

Mathematically, an aggregate function $f$ takes a collection of values $X = \{x_1, x_2, \dots, x_n\}$ and returns a single value. Common examples include:

- **Mean (Average):**
    
    $$\bar{x} = \frac{1}{n} \sum_{i=1}^{n} x_i$$
    
- **Count:** The total number of non-null items in the group.
    
- **Variance/Standard Deviation:** Useful for understanding the spread of your data.

| **Function**     | **PySpark SQL Function** | **Description**                              |
| ---------------- | ------------------------ | -------------------------------------------- |
| **Sum**          | `F.sum()`                | Adds all values in the group.                |
| **Average**      | `F.avg()`                | Calculates the arithmetic mean.              |
| **Count**        | `F.count()`              | Counts rows (or non-null values).            |
| **Min/Max**      | `F.min()` / `F.max()`    | Finds the boundary values.                   |
| **Collect List** | `F.collect_list()`       | Returns an array of all values in the group. |

## Important Tips

- **The "GroupedData" Object:** When you call `df.groupBy("col")`, Spark doesn't return a DataFrame immediately. It returns a `GroupedData` object. You **must** call an aggregate function to turn it back into a DataFrame.
    
- **Performance:** Grouping causes a **Shuffle**. This means data is moved across the cluster so that all rows for "Sales" end up on the same worker node. If you have millions of unique groups, this can be resource-intensive.
    
- **Filtering:** If you want to filter results _after_ grouping (the SQL equivalent of `HAVING`), just use a standard `.filter()` or `.where()` on the resulting DataFrame.