---
up:
  - "[[003_skills/data-engineering/06 Spark/0603 SQL/060303 Advance operator/060303 Advance operator|060303 Advance operator]]"
down:
prev:
topic: false
question: What are Pandas UDF? How to use it?
---
# What are Pandas UDF?


> [!Summary] Summary
> - It is similar to UDF. It uses vectored pandas datatype hence performs better than UDF 
> - Instead of using row-by-row data it uses vectored pandas datatype using Apache arrow 
> - This is preferred when there is no inbuilt spark function available. It can be used for AI/ML packages.

**Pandas UDFs** (also known as **Vectorized UDFs**) are a high-performance alternative to standard Python UDFs in PySpark. Introduced in Spark 2.3, they allow you to perform distributed computations using **Pandas** and **NumPy** logic, while leveraging **Apache Arrow** for extremely fast data transfer.

## Why were they created? 
Standard Python UDFs are notoriously slow because they process data **one row at a time**.
1. Spark has to serialize each row into a format Python understands.
2. It sends that row to a Python worker.
3. The Python worker processes it and sends it back.
This "row-by-row" overhead is a massive bottleneck. **Pandas UDFs** solve this by processing data in **batches**.

## The Secret Sauce: Apache Arrow
The efficiency of Pandas UDFs comes from **Apache Arrow**. Arrow is a cross-language, columnar memory format.
- Instead of converting data row-by-row, Spark uses Arrow to move entire chunks (batches) of data from the JVM to the Python worker memory.
- Once in Python, the data is represented as a **Pandas Series** or **DataFrame**, which allows you to use highly optimized vectorized operations (like those in NumPy).

## Common Types of Pandas UDFs
The syntax for Pandas UDFs relies on **Python Type Hints** to tell Spark how to handle the input and output.
### 1. Series to Series (Scalar)
This is the most common type. It takes a Pandas Series as input and returns a Pandas Series of the same length.
```python
import pandas as pd
from pyspark.sql.functions import pandas_udf

# Input and output are both pandas.Series
@pandas_udf("double")
def multiply_by_ten(s: pd.Series) -> pd.Series:
    return s * 10

df.withColumn("multiplied", multiply_by_ten(df["price"]))
```

### 2. Iterator of Series to Iterator of Series

This is useful for logic that requires **state initialization** (like loading a machine learning model once and then using it to predict on multiple batches of data).

```python
from typing import Iterator

@pandas_udf("double")
def predict(batch_iter: Iterator[pd.Series]) -> Iterator[pd.Series]:
    # Load model once here (Expensive initialization)
    model = load_my_model() 
    for s in batch_iter:
        yield model.predict(s)
```

### 3. Grouped Map (`applyInPandas`)
This allows you to group your Spark DataFrame by a key and then apply a function that takes a full **Pandas DataFrame** and returns another **Pandas DataFrame**. This is popular for training a separate model for every user or city in your dataset.

```python
def subtract_mean(pdf: pd.DataFrame) -> pd.DataFrame:
    # pdf is a pandas.DataFrame
    v = pdf.v
    return pdf.assign(v=v - v.mean())

# Used via the .groupby().applyInPandas() method
df.groupby("id").applyInPandas(subtract_mean, schema="id long, v double")
```

## Performance Comparison

|**Feature**|**Standard UDF**|**Pandas UDF**|
|---|---|---|
|**Data Processing**|Row-by-row|**Vectorized (Batched)**|
|**Data Transfer**|Pickle Serialization (Slow)|**Apache Arrow (Fast)**|
|**Optimization**|Black Box (No optimization)|Limited but faster execution|
|**Best For**|Simple if/else logic|**Complex math, ML, or Statistics**|

## When to use Pandas UDFs?

1. **When no native Spark function exists:** If you can do it with `F.col("a") + F.col("b")`, do it! Native is always faster.
2. **Machine Learning Inference:** When you want to run a Scikit-learn or TensorFlow model over a Spark DataFrame.
3. **Complex Statistics:** When you need a function that exists in Pandas/SciPy but not in Spark SQL.
