---
up:
  - "[[000_+/060302 Operations|060302 Operations]]"
down:
prev:
topic: false
question: What are SQL action operations?
---
# What are SQL action operations?


> [!Summary] Summary
> Contents

|**Category**|**Action / Method**|**Primary Purpose**|**Key Note / Warning**|
|---|---|---|---|
|**Data Retrieval**|`show(n)`|Displays the first $n$ rows in the console.|Great for debugging; default $n$ is 20.|
||`collect()`|Returns the **entire** dataset to the driver.|**Danger:** Can cause `OutOfMemoryError` on large data.|
||`first()` / `head(n)`|Returns the very first row (or $n$ rows) as objects.|Useful for inspecting schema/values in code.|
||`take(n)`|Fetches the first $n$ rows to the driver.|Safer than `collect()` for a quick peek at data.|
|**Stats & Math**|`count()`|Returns the total number of rows.|Often used to verify filter results or data loads.|
||`describe()` / `summary()`|Provides mean, stddev, min, and max.|Essential for initial Exploratory Data Analysis (EDA).|
||`reduce(func)`|Aggregates data using a specific binary function.|Combines all rows into a single result (e.g., total sum).|
|**Export & Save**|`write`|Saves data to files (CSV, Parquet, etc.) or tables.|The primary way to persist results to disk.|
||`foreach(func)`|Runs a function on every single row.|Typically used for side effects, like calling an external API.|

|**Feature**|**Transformations (e.g., select, filter)**|**Actions (e.g., count, save)**|
|---|---|---|
|**Execution**|Lazy (builds a plan)|Eager (starts execution)|
|**Output**|Returns a new DataFrame|Returns a result or saves data|
|**Example**|`df.where(col("age") > 21)`|`df.count()`|


- **Actions** are the operations that trigger the actual execution of the lazy transformations you've defined. 
- Until you call an action, Spark simply builds a logical plan (the DAG). 
- When an action is invoked, Spark sends the tasks to the cluster executors to compute the result.

Here is a breakdown of the most common Spark SQL actions:

## 1. Data Retrieval Actions
These operations pull data from the executors back to the driver program or display it to the console.

- **`show(n)`**: Displays the first $n$ rows in a tabular format. It is the most common tool for debugging and data inspection.
- **`collect()`**: Returns the entire dataset to the driver as an array/list.
    > **Warning:** Use this cautiously. If the dataset is larger than the driver's memory, it will cause an `OutOfMemoryError`.
- **`first()` / `head(n)`**: Returns the first row or the first $n$ rows of the DataFrame.
- **`take(n)`**: Similar to `head`, it fetches the first $n$ rows and returns them to the driver.

## 2. Computational & Statistical Actions
These actions perform a calculation over the entire dataset and return a single value or a summary.
- **`count()`**: Returns the total number of rows in the DataFrame.
- **`describe()` / `summary()`**: Computes basic statistics (mean, standard deviation, min, max) for numerical columns.
- **`reduce()`**: Aggregates the elements of the DataFrame using a binary function (e.g., summing all values in a column).

## 3. Export & Persistence Actions
These actions trigger the processing of data to store it permanently or move it outside of Spark.
- **`write`**: This is the gateway to saving data. Combined with formats, it becomes an action:
    - `df.write.csv("path")`
    - `df.write.parquet("path")`
    - `df.write.saveAsTable("table_name")`
- **`foreach(func)`**: Applies a function to each row of the DataFrame. This is often used to send data to an external database or API.

## Why "Actions" Matter
- Because of **Lazy Evaluation**, Spark can optimize your entire query before running it. 
- For example, if you filter a dataset and then call `count()`, Spark’s optimizer (Catalyst) might decide to filter the data at the data source level (Predicate Pushdown) rather than loading everything into memory first. 
- This optimization only happens once an **Action** is called.

