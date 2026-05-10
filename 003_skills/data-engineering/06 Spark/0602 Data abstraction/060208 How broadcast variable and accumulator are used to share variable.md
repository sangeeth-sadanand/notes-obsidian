---
up:
  - "[[003_skills/data-engineering/06 Spark/0602 Data abstraction/0602 Data abstraction|0602 Data abstraction]]"
down:
prev:
topic: false
question: How broadcast variable and accumulator are used to share variable?
---
# How broadcast variable and accumulator are used to share variable?

> [!Summary] Summary
> - A broadcast variable is used to share value from driver to the executor 
> - While an accumulator is used to share value from executor to drivers 
> - broadcast variable are read-only and accumulators are write-only
> 
> | **Feature**     | **Broadcast Variables**            | **Accumulators**                   |
> | --------------- | ---------------------------------- | ---------------------------------- |
> | **Direction**   | Driver $\to$ Executors             | Executors $\to$ Driver             |
> | **Access**      | Read-Only (on Executors)           | Write-Only (on Executors)          |
> | **Primary Use** | Efficiently sharing large datasets | Global counters or sums            |
> | **Update Rule** | Static once created                | Commutative (order doesn't matter) |
> 

- In Apache Spark, **Broadcast variables** and **Accumulators** are the two primary ways to share data between the driver program and the executors. 
- Since Spark is a distributed system, standard variables aren't automatically shared across the cluster in an efficient way; these two tools solve that in very different directions.

## **1. Broadcast Variables (Read-Only)**
Broadcast variables allow the programmer to keep a **read-only** variable cached on each machine rather than shipping a copy of it with tasks. They are used to give every node a large input dataset efficiently.
- **The Problem:** Normally, if you use a large variable (like a lookup table) in a map function, Spark sends that variable to the executors **every time** a task is launched. This wastes network bandwidth.
- **The Solution:** Spark distributes the broadcast variable to each **node** only once using efficient, P2P-like gossip protocols.

### **Common Use Case: Side-Input Joins**

If you have a massive table of "Transactions" and a small table of "User Metadata," you "broadcast" the small table to all nodes so they can perform a local join without shuffling the massive table.
```python hl=4
# Create a lookup table
country_map = {"US": "United States", "UK": "United Kingdom"}
# Broadcast it to the cluster
broadcast_countries = sc.broadcast(country_map)

# Use it inside a transformation
rdd.map(lambda x: (x[0], broadcast_countries.value.get(x[1])))
```

## **2. Accumulators (Write-Only/Counters)**
Accumulators are variables that are only "added" to through an associative and commutative operation. They are used to implement **counters** or **sums** across the cluster.
- **The Problem:** If you try to increment a standard counter variable inside a `map()` function, each executor increments its own local copy. The driver's version of that variable remains unchanged.
- **The Solution:** Accumulators provide a way to send updates back to the driver.
    
### **Common Use Case: Error Logging or Data Quality**
You might use an accumulator to count how many "corrupt" records you encounter during a massive data processing job.

```python
# Initialize the accumulator on the driver
blank_lines = sc.accumulator(0)

def count_blanks(line):
    global blank_lines
    if line == "":
        blank_lines += 1
    return line

rdd.map(count_blanks).collect()

# Only the driver can read the value
print(f"Blank lines found: {blank_lines.value}")
```


