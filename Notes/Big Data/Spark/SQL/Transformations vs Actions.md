# Transformations vs Actions
## Transformations

- **Definition:** Operations that create a new RDD/DataFrame from an existing one.
- **Lazy Evaluation:** They don’t execute immediately; Spark records them in a Directed Acyclic Graph (DAG).
- **Examples:**
    - `map()` – apply a function to each element
    - `filter()` – keep only rows matching a condition
    - `select()` – choose specific columns
    - `join()` – combine two datasets
    - `groupBy()` – group rows by key
### When You Call a Transformation

- Spark **does not run immediately**.
- Instead, it **records the operation** (like `map`, `filter`, `select`) in its **DAG (Directed Acyclic Graph)**.
- Think of it like writing down a recipe: you’re describing _what should be done_, but you haven’t started cooking yet.
- Example:
    
    ```python
    df2 = df.filter(df.age > 30)
    ```
    
    Here, Spark just notes: “Filter rows where age > 30.” No computation happens yet.


## Actions

- **Definition:** Operations that trigger execution and return results to the driver or write to storage.
- **Execution Trigger:** Spark runs the DAG only when an action is called.
- **Examples:**
    - `collect()` – return all elements to the driver
    - `count()` – number of rows
    - `show()` – display rows in console
    - `saveAsTextFile()` – write results to disk
    - `reduce()` – aggregate values
###  When You Call an Action

- Spark says: “Okay, now I need actual results.”
- It **triggers execution** of the DAG built from all prior transformations.
- Spark:
    1. Optimizes the DAG (removes redundant steps, reorders operations).
    2. Splits the work into **tasks** across the cluster.
    3. Executes tasks in parallel.
    4. Returns results to the driver (for actions like `collect`, `count`) or writes them to storage.
- Example:
    
    ```python
    df2.show()
    ```
    
    This forces Spark to run the filter, distribute tasks, and finally display the rows.
