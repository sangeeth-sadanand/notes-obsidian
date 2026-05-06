---
up:
  - "[[003_skills/data-engineering/060303 Advance operator|060303 Advance operator]]"
down:
prev:
topic: false
question: How do pivot operations reshape data frame?
---
# How do pivot operations reshape data frame?


> [!Summary] Summary
> - Pivot are used to rotate data perspective. It transforms **long** data frame to **wide** data frame 
> - To pivot a dataframe we need to define three elements: 
> 	1. Grouping columns 
> 	2. Pivot columns 
> 	3. Aggregate function 
> - if a pivot element does not have a value in a given row for a column then it insert `na` 
> - It is recommended to provide pivot values otherwise it will eagerly compute the values at runtime causing 2 time computation of data.

- Pivoting is essentially a rotation of your data's perspective. It transforms a **long** dataset (where variables are stored in rows) into a **wide** dataset (where variables become distinct columns).

## The Transformation Logic
To perform a pivot in PySpark, you generally need three ingredients:
1. **Grouping Column(s):** These stay as rows (your Y-axis).
2. **Pivot Column:** The values in this column will become your new column headers (your X-axis).
3. **Aggregation:** Since multiple rows might collapse into one cell, you must tell Spark how to handle the data (e.g., `sum`, `avg`, `count`).

### Visualizing the Shift
Imagine you have a sales table:

|**Date**|**Product**|**Sales**|
|---|---|---|
|2023|Apple|100|
|2023|Orange|150|
|2024|Apple|200|
|2024|Orange|250|

**After pivoting on "Product":**

|**Date**|**Apple**|**Orange**|
|---|---|---|
|2023|100|150|
|2024|200|250|

## Implementing it in PySpark

The syntax follows a specific chain: `groupBy` → `pivot` → `agg`.

```python
from pyspark.sql import functions as F

# 1. Group by the column you want to keep as rows
# 2. Pivot the column you want to turn into headers
# 3. Aggregate the values to fill the new columns
pivoted_df = df.groupBy("Date") \
               .pivot("Product") \
               .agg(F.sum("Sales"))
```

### Pro-Tip: Efficiency
PySpark is "lazy," but pivoting is one of the few operations that can be quite "eager" and expensive. By default, Spark has to scan your entire dataset to find all unique values in the pivot column to create the new headers.
If you already know the unique values, you can pass them as a list to the `pivot()` function to speed things up significantly:

```python
products = ["Apple", "Orange"]
df.groupBy("Date").pivot("Product", products).agg(F.sum("Sales"))
```

## Important Considerations
- **Null Handling:** If a specific group doesn't have a value for one of the pivoted columns, Spark will fill it with `null`. You can chain `.na.fill(0)` afterward if you prefer zeros.
- **Cardinality:** Be careful! Pivoting a column with thousands of unique values (like "User ID") will create thousands of columns, which can lead to memory issues or even crash your Spark driver. It’s best suited for columns with low cardinality (like "Months," "Regions," or "Categories").
- **The Opposite:** If you ever need to go back from wide to long, you’ll want to look into the `stack` function or `melt` operations.
