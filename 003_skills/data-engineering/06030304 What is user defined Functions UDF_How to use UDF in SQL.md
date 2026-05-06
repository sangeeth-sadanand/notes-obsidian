---
up:
  - "[[003_skills/data-engineering/060303 Advance operator|060303 Advance operator]]"
down:
prev:
topic: false
question: What is user defined Functions UDF? How to use UDF in SQL?
---
# What is user defined Functions UDF? How to use UDF in SQL?


> [!Summary] Summary
> - UDF enables to define a new python function in spark that is not available in spark. 
> - We can use machine learning function of python in spark. 
> - Spark does not apply optimization on UDF 
> - We should avoid UDF if there is already existing function in spark



A **User-Defined Function (UDF)** is a feature in Spark that allows you to extend the vocabulary of the Spark SQL engine. If Spark’s built-in functions (like `sum`, `count`, or `regexp_extract`) don't quite cut it for your specific logic, you can write a custom function in Python and "wrap" it so Spark can use it on your DataFrames.

## How UDFs Work (Under the Hood)
In PySpark, there is a catch: Spark runs on the **Java Virtual Machine (JVM)**, but your UDF is written in **Python**. For Spark to run your code, it has to move data back and forth between these two environments.
1. **Serialization:** Spark takes the data from the JVM and "pickles" it (serializes it) to a format Python understands.
2. **Transmission:** The data is sent to a separate Python worker process.
3. **Execution:** Your Python function runs on the data.
4. **Return:** The result is serialized again and sent back to the JVM.

## Basic UDF Example
Here is how you would create a simple UDF to convert a string to "Leet Speak":
```python
from pyspark.sql import functions as F
from pyspark.sql.types import StringType

# 1. Define a standard Python function
def to_leet(text):
    if text is None: return None
    return text.replace('e', '3').replace('a', '4').replace('s', '5')

# 2. Register it as a Spark UDF
leet_udf = F.udf(to_leet, StringType())

# 3. Use it on a DataFrame
df = df.withColumn("leet_name", leet_udf(F.col("name")))
```

## The "Black Box" Performance Problem
Standard UDFs come with a performance warning. Because your logic is hidden inside a Python function, Spark’s **Catalyst Optimizer** (the brain that makes Spark fast) treats it as a "Black Box."
- **No Optimization:** Spark can't see inside your Python code to optimize it.
- **Overhead:** The constant moving of data between the JVM and Python is slow.
- **Row-by-Row:** Standard UDFs process data one row at a time, which is inefficient for large datasets.

### The 3-Step Process to use UDF in SQL

#### 1. Define the Python Logic
First, create the standard Python function that contains your logic.
```python
def calculate_vat(price):
    if price is None:
        return 0.0
    return price * 1.20
```

#### 2. Register the Function
Use `spark.udf.register()` to make it available to the SQL engine. This requires two things: a **name** (how you'll call it in SQL) and the **function** itself.
```python
# Registration syntax: (name_in_sql, python_function, return_type)
spark.udf.register("get_vat", calculate_vat, "double")
```

#### 3. Use it in an SQL Query
Now that it's in the registry, you can use it just like any built-in SQL function (like `UPPER()` or `SUM()`).
```python
# You can now call 'get_vat' directly inside a string query
df_results = spark.sql("""
    SELECT 
        product_id, 
        price, 
        get_vat(price) as price_with_tax 
    FROM sales_table
""")

df_results.show()
```

### Important Details to Remember
- **Scope:** Registration is **session-scoped**. If you create a new Spark session, you will need to register the UDF again to use it in that new session.
- **The Return Type:** It is critical to specify the return type (e.g., `StringType()`, `IntegerType()`, or a string shorthand like `"double"`) during registration. If you don't, Spark defaults to `StringType`, which might break your math operations later.
- **Registration Returns a Reference:** The `spark.udf.register` method actually returns a UDF object. This means you can use the same registration for both the DataFrame API and SQL:
```python
sql_vat = spark.udf.register("get_vat", calculate_vat, "double")

# Works in SQL
spark.sql("SELECT get_vat(price) FROM table")

# Works in DataFrame API
df.withColumn("tax", sql_vat("price"))
```
    
### When should you avoid this?

Just like standard UDFs, SQL-registered UDFs still force Spark to move data from the JVM to Python. If you can achieve your goal using **Spark SQL built-in functions** (like `CASE WHEN`, `COALESCE`, or `EXPLODE`), your query will run significantly faster because it stays entirely within the optimized JVM environment.
