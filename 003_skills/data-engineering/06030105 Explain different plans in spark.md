---
up:
  - "[[003_skills/data-engineering/060301 Fundamentals|060301 Fundamentals]]"
down:
prev:
topic: false
question: Explain different plans in spark?
---
# Explain different plans in spark?


> [!Summary] Summary
> 
> Spark SQL transition from a user writen query to actual execution plan in 4 steps 
> - **Unresolved logical plan**: here the SQL or dataframe code is parsed, and spark checks for basic syntax 
> - **Resolved logical plan**: In this spark uses catalog data to validate table, column names and column data types to the unresolved plan 
> - The catalyst optimizer applies set of rules to optimize the plan this result is **optimized logical plan** 
> - Using file location and partition information optimized plan is converted to **physical plan** 
> 
> | **Plan Stage**         | **Purpose**             | **Key Action**                                 |
> | ---------------------- | ----------------------- | ---------------------------------------------- |
> | **Unresolved Logical** | Syntax Check            | Parsed from SQL/DataFrame code.                |
> | **Resolved Logical**   | Semantic Check          | Column/Table names validated against Catalog.  |
> | **Optimized Logical**  | Rule-Based Optimization | Predicate pushdown and column pruning applied. |
> | **Physical Plan**      | Execution Strategy      | Choosing join algorithms and partitioning.     |
> 

 - In Spark SQL, the transition from a user-written query to the actual execution on a cluster involves a series of transformations. 
 - These transformations are managed by the **Catalyst Optimizer**, which generates several "plans" to ensure the data is processed as efficiently as possible.

## 1. Logical Plans
A logical plan describes **what** you want to do without specifying **how** it will be done physically. It abstracts away the underlying storage and computing details.
- **Unresolved Logical Plan:** This is the first step after parsing your SQL or DataFrame code. Spark checks for basic syntax but hasn't yet verified if the columns or tables actually exist in the metadata.
- **Resolved Logical Plan:** Spark consults the **Catalog** (a repository of metadata) to validate table names and column types. At this stage, "unresolved" attributes become "resolved."
- **Optimized Logical Plan:** The Catalyst Optimizer applies a set of rule-based optimizations. For example, it might perform **Constant Folding** (calculating $2 + 2$ as $4$ before running) or **Predicate Pushdown** (filtering data as early as possible to reduce the volume of data moved).

## 2. Physical Plans
Once the logical plan is optimized, Spark moves into the physical planning phase. This describes **how** the execution will actually happen on the cluster hardware.
- **Physical Plan Generation:** Spark generates multiple physical strategies for a single logical plan. For example, if you are joining two tables, it might consider a **Broadcast Hash Join** versus a **Sort Merge Join**.
- **Cost-Based Model (CBB):** Spark evaluates these strategies based on "cost" (estimated execution time, memory usage, and I/O) and selects the most efficient one.
- **Selected Physical Plan:** This is the specific version chosen for execution. It is further broken down into **Stages** and **Tasks** that are scheduled across the executors.

## How to View These Plans
You can see this process in action by calling the `.explain(True)` method on any DataFrame.

```python
df.filter("age > 21").select("name").explain(True)
```

**What to look for in the output:**
- **Parsed Logical Plan:** Your raw query structure.
- **Analyzed Logical Plan:** The plan with data types confirmed.
- **Optimized Logical Plan:** The "slimmed down" version of your logic.
- **Physical Plan:** The final blueprint, often showing markers like `*(1) Filter` or `*(1) Project`, which indicate **Whole-Stage Code Generation** is active.
