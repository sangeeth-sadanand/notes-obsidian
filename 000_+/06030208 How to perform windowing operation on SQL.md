---
up:
  - "[[000_+/060302 Operations|060302 Operations]]"
down:
prev:
topic: false
question: How to perform windowing operation on SQL?
---
# How to perform windowing operation on SQL?


> [!Summary] Summary
> Contents


- Window functions are one of the most powerful tools in SQL. 
- Unlike standard aggregate functions (like `SUM` or `AVG`), which collapse multiple rows into a single result, **window functions perform a calculation across a set of table rows that are somehow related to the current row.**

## 1. The Basic Syntax
The hallmark of a window function is the `OVER()` clause. This clause defines the "window" of rows the function will look at.

```sql
SELECT 
    column_name,
    FUNCTION_NAME(column_name) OVER (
        PARTITION BY column_to_group
        ORDER BY column_to_sort
        ROWS BETWEEN ... -- Optional: Defines the frame
    ) as alias_name
FROM table_name;
```

```python
from pyspark.sql import Window
from pyspark.sql import functions as F

# 1. Define the Window Specification (the "OVER" clause)
windowSpec = Window \
    .partitionBy("column_to_group") \
    .orderBy("column_to_sort") \
    .rowsBetween(start_boundary, end_boundary) # Optional: The "ROWS BETWEEN" part

# 2. Apply the Window Function
df_result = df.withColumn("alias_name", F.function_name("column_name").over(windowSpec))
```
### The Three Pillars of `OVER()`:
- **`PARTITION BY`**: Acts like a `GROUP BY` but doesn't collapse rows. It divides the data into logical buckets (e.g., partitioning by "Department").
- **`ORDER BY`**: Defines the sequence of rows within each partition. This is vital for functions like "Running Totals" or "Ranking."
- **`ROWS/RANGE` (The Frame)**: Defines how many rows before or after the current row to include in the calculation.

## 2. Common Window Functions

### A. Aggregate Window Functions
These are your standard aggregates used within a window.
- **Running Total:** Calculating the cumulative sum of sales over time.
```sql
SELECT 
    SaleDate, 
    Amount,
    SUM(Amount) OVER (ORDER BY SaleDate) AS RunningTotal
FROM Sales;
```

### B. Ranking Functions
Perfect for "Top N" lists or identifying duplicates.

|**Function**|**Behavior**|
|---|---|
|**`ROW_NUMBER()`**|Assigns a unique, sequential integer (1, 2, 3...).|
|**`RANK()`**|Assigns rank; skips numbers if there's a tie (1, 2, 2, 4).|
|**`DENSE_RANK()`**|Assigns rank; no gaps after ties (1, 2, 2, 3).|

### C. Value (Positional) Functions
These allow you to "look" at other rows relative to the current one.
- **`LAG()`**: Returns the value from a previous row (useful for calculating month-over-month growth).
- **`LEAD()`**: Returns the value from a subsequent row.

## 3. Window Frames: `ROWS BETWEEN`

If you want to calculate a **3-day moving average**, you need to define a frame. This tells SQL exactly which rows "surrounding" the current row to include.

```
SELECT 
    Date, 
    Sales,
    AVG(Sales) OVER (
        ORDER BY Date 
        ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING
    ) AS ThreeDayMovingAvg
FROM DailyRevenue;
```

### The Boundary Keywords
- **`UNBOUNDED PRECEDING`**: The very first row of the partition.
- **`n PRECEDING`**: A physical number of rows ($n$) before the current row.
- **`CURRENT ROW`**: The row currently being processed.
- **`n FOLLOWING`**: A physical number of rows ($n$) after the current row.
- **`UNBOUNDED FOLLOWING`**: The very last row of the partition.