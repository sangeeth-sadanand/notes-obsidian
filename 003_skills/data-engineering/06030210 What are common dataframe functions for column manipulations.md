---
up:
  - "[[003_skills/data-engineering/060302 Operations|060302 Operations]]"
down:
prev:
topic: false
question: What are common dataframe functions for column manipulations?
---
# What are common dataframe functions for column manipulations?


> [!Summary] Summary
> 
> |Category|Function Examples|Use Case|
> |---|---|---|
> |Selection|`col`, `lit`, `expr`, `withColumn`|Access or create columns|
> |Conditional|`when`, `coalesce`, `isnull`, `nanvl`|Handle nulls, apply logic|
> |String|`concat`, `split`, `regexp_replace`, `trim`|Text cleaning & formatting|
> |Math|`abs`, `round`, `sqrt`, `pow`, `greatest`|Numeric transformations|
> |Array/Map|`array`, `explode`, `create_map`|Complex data structures|
> |Date/Time|`current_date`, `datediff`, `year`, `month`|Time-based analysis|
> 

 
## 1. **Column Selection & Creation**

- **`col("column_name")`** – Access a column by name.
- **`lit(value)`** – Create a column with a literal value.
- **`expr("expression")`** – Use SQL-like expressions directly.
- **`withColumn("new_col", function)`** – Add or replace a column.

```python 
df.select(F.col("name")).show()                 # col
df.select(F.lit("constant").alias("lit_col")).show()  # lit
df.select(F.expr("score * 2").alias("expr_col")).show() # expr
df.withColumn("score_plus_10", F.col("score") + 10).show() # withColumn
```

## 2. **Conditional & Null Handling**

- **`when(condition, value)`** – Conditional logic (like SQL CASE).
- **`coalesce(col1, col2, ...)`** – Return first non-null value.
- **`isnull(col)` / `isnotnull(col)`** – Check for nulls.
- **`nanvl(col1, col2)`** – Replace NaN with another column’s value.

```python 
df.withColumn("pass_fail", F.when(F.col("score") >= 60, "pass").otherwise("fail")).show()
df.withColumn("score_filled", F.coalesce(F.col("score"), F.lit(0))).show()
df.withColumn("is_null", F.col("score").isNull()).show()
df.withColumn("is_not_null", F.col("score").isNotNull()).show()
df.withColumn("nanvl_demo", F.nanvl(F.lit(float("nan")), F.lit(100))).show()

```

## 3. **String Manipulations**

- **`concat(col1, col2, ...)`** – Concatenate strings.
- **`split(col, pattern)`** – Split string into array.
- **`regexp_replace(col, pattern, replacement)`** – Regex-based replacement.
- **`lower(col)` / `upper(col)`** – Case transformations.
- **`trim(col)`** – Remove whitespace.

```python
df.withColumn("concat_demo", F.concat(F.col("name"), F.lit("_test"))).show()
df.withColumn("split_demo", F.split(F.col("name"), "a")).show()
df.withColumn("regex_demo", F.regexp_replace(F.col("name"), "li", "LI")).show()
df.withColumn("lower_demo", F.lower(F.col("name"))).show()
df.withColumn("upper_demo", F.upper(F.col("name"))).show()
df.withColumn("trim_demo", F.trim(F.col("name"))).show()

```

## 4. **Math & Numeric Functions**

- **`abs(col)`**, **`round(col, n)`**, **`sqrt(col)`**, **`pow(col, n)`** – Standard math operations.
- **`greatest(col1, col2, ...)`** / **`least(col1, col2, ...)`** – Compare multiple columns.
- **`rand()` / `randn()`** – Generate random numbers.

```python
df.withColumn("abs_demo", F.abs(F.col("score"))).show()
df.withColumn("round_demo", F.round(F.col("score"), 0)).show()
df.withColumn("sqrt_demo", F.sqrt(F.col("score"))).show()
df.withColumn("pow_demo", F.pow(F.col("score"), 2)).show()
df.withColumn("greatest_demo", F.greatest(F.col("score"), F.lit(60))).show()
df.withColumn("least_demo", F.least(F.col("score"), F.lit(60))).show()
df.withColumn("rand_demo", F.rand()).show()
df.withColumn("randn_demo", F.randn()).show()

```

## 5. **Array & Map Functions**

- **`array(col1, col2, ...)`** – Create array column.
- **`explode(array_col)`** – Flatten arrays into rows.
- **`create_map(col1, col2, ...)`** – Create map column.

```python
df.withColumn("array_demo", F.array(F.col("id"), F.col("score"))).show()
df.select("id", F.explode("items").alias("item")).show()
df.withColumn("map_demo", F.create_map(F.lit("id"), F.col("id"))).show()

```
## 6. **Date & Time Functions**

- **`current_date()` / `current_timestamp()`** – Current date/time.
- **`datediff(col1, col2)`** – Difference in days.
- **`date_add(col, days)` / `date_sub(col, days)`** – Add/subtract days.
- **`year(col)` / `month(col)` / `dayofmonth(col)`** – Extract date parts.

```python
df.withColumn("current_date_demo", F.current_date()).show()
df.withColumn("current_ts_demo", F.current_timestamp()).show()
df.withColumn("datediff_demo", F.datediff(F.current_date(), F.col("date"))).show()
df.withColumn("date_add_demo", F.date_add(F.col("date"), 7)).show()
df.withColumn("date_sub_demo", F.date_sub(F.col("date"), 7)).show()
df.withColumn("year_demo", F.year(F.col("date"))).show()
df.withColumn("month_demo", F.month(F.col("date"))).show()
df.withColumn("day_demo", F.dayofmonth(F.col("date"))).show()
```
