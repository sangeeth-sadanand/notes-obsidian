---
up:
  - "[[003_skills/data-engineering/060301 Fundamentals|060301 Fundamentals]]"
down:
prev:
topic: false
question: What are the fundamental principles of spark SQL?
---
# What are the fundamental principles of spark SQL?


> [!Summary] Summary
> 
> The fundamental principles of spark 
> - **Dataframe abstraction**: data frame a distributed collection of data object organised into named columns conceptually equivalent to table 
> - **Catalyst optimizer**: It optimizes the query for performance, it analyses the datatype, column name and table with the catalog, it does constant folding, predicate pushdown and column pruning
> - **Tungsten execution engine** - optimizes the hardware effectively by using off heap memory, Binary file processing, and collapsing multiples operation into one function for code locality
> - **Unified data access** - allows data from files, or database all as dataframes 


Spark SQL is designed to bridge the gap between big data processing and traditional relational databases.

## 1. The DataFrame and Dataset Abstraction
The core of Spark SQL is the **DataFrame**, a distributed collection of data organized into named columns. Conceptually, it is equivalent to a table in a relational database but with richer optimizations under the hood.
- **DataFrames:** Untyped (generic `Row` objects), ideal for Python and R.
- **Datasets:** Strongly typed, available in Scala and Java, providing compile-time type safety.

## 2. The Catalyst Optimizer
This is the "brain" of Spark SQL. Catalyst is a sophisticated extensible optimization framework based on functional programming constructs in Scala. It automates the process of turning a query into an efficient physical execution plan through four main phases:
1. **Analysis:** Resolving column and table names against the Catalog.
2. **Logical Optimization:** Applying rule-based optimizations like **constant folding**, **predicate pushdown**, and **column pruning**.
3. **Physical Planning:** Generating multiple physical plans and choosing the most efficient one using a cost model.
4. **Code Generation:** Compiling the final plan into Java bytecode for execution.

## 3. Tungsten Execution Engine
While Catalyst optimizes the _logic_, Tungsten optimizes the _hardware_ efficiency. It focuses on:
- **Memory Management:** Using an "off-heap" memory management layout to avoid Java Garbage Collection (GC) overhead.
- **Binary Processing:** Operating directly on binary data rather than deserializing it into Java objects.
- **Whole-Stage Code Generation:** Collapsing multiple operators into a single function to improve CPU cache locality and reduce virtual function calls.

## 4. Unified Data Access
- Spark SQL provides a single interface to interact with a vast array of data sources. 
- Whether the data is in **JSON, Parquet, Avro, Hive, or a JDBC-connected SQL database**, Spark SQL treats them all as DataFrames. This allows you to join a Parquet file with a MySQL table in a single SQL statement.

## 5. Standard Connectivity (Thrift Server)
- Spark SQL supports industry-standard connectivity through its **JDBC/ODBC server**. 
- This allows business intelligence (BI) tools like Tableau, Power BI, or Looker to connect to Spark and execute SQL queries just as they would with a traditional database like Postgres or SQL Server.

