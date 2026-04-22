---
up:
  - "[[000_+/0602 Data abstraction|0602 Data abstraction]]"
down:
prev:
topic: false
question: What is RDD?
---
# What is RDD?


> [!Summary] Summary
> Contents


In **Apache Spark**, an **RDD** stands for **Resilient Distributed Dataset**.  
It is the **core, low-level data structure** of Spark used for distributed data processing.
An **RDD** is a **fault-tolerant, immutable collection of data elements** that is:
- **Distributed** across multiple machines (nodes)
- **Processed in parallel**
- **Resilient** to failures

## Key Characteristics of RDD

### 1. **Resilient**
- Spark automatically **recovers lost data** if a node fails
- Uses **lineage** (how data was created) to recompute lost partitions

### 2. **Distributed**
- Data is split into **partitions**
- Each partition is processed on different executors

### 3. **Immutable**
- Once created, an RDD **cannot be changed**
- Any modification creates a **new RDD**

### 4. **Lazy Evaluation**
- Transformations are **not executed immediately**
- Execution happens only when an **action** is called

Use RDDs when:
- You need **fine-grained control**
- Working with **unstructured data**
- Performance tuning at low level
- Using complex custom logic not supported by DataFrames


