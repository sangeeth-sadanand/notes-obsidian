---
up:
  - "[[003_skills/data-engineering/06 Spark/0603 SQL/060304 Optimization/060304 Optimization|060304 Optimization]]"
down:
prev:
topic: false
question: Memory tuning parameters
---
# Memory tuning parameters
> [!Summary] Summary
> 
> - **spark.executor.memory**-JVM heap memory 
> - **spark.executor.memoryOverhead** -[10%] (minimum 3841913) 
> - **spark.memory.fraction** [0.6]- fraction for execution + storage memory (spark memory) 
> - **spark. memory**-storage Fraction [0.5]-fraction for storage memory in spark memory 
> - **spark.memory.offHeap.enable** - to enable off heap 
> - **spark.memory. offHeap.size** - to give off heap memory size

Tuning Apache Spark’s memory is largely about balancing the needs of your specific application—whether it's caching heavily, performing massive shuffles, or running complex User-Defined Functions (UDFs).
Here are the most critical tuning parameters you can adjust to optimize memory allocation and prevent OutOfMemory (OOM) errors.

## **1. Core Memory Allocation**
These parameters dictate the absolute amount of memory provisioned by the cluster manager (like YARN or Kubernetes) for your Spark application.
- **`spark.executor.memory`** (Default: `1g`)
    - **What it does:** The amount of JVM heap memory allocated to each executor.
    - **Tuning tip:** Increase this for applications handling large datasets per partition. However, avoid setting it too high (e.g., >64GB) as it can lead to massive Garbage Collection (GC) pauses. If you need more memory, it's often better to scale horizontally by adding more executors.
- **`spark.executor.memoryOverhead`** (Default: `10%` of executor memory, minimum `384m`)
    - **What it does:** This is off-heap memory allocated by the cluster manager for VM overhead, interned strings, NIO direct-buffer allocations, and PySpark workers (if you are using Python).
    - **Tuning tip:** If your application is getting killed by the cluster manager (e.g., YARN throwing a "Container killed by YARN for exceeding memory limits" error), increase this value (e.g., to `spark.executor.memoryOverhead=1g` or higher).


## **2. Internal Heap Partitioning**
These parameters control how the JVM heap (`spark.executor.memory`) is divided internally.
- **`spark.memory.fraction`** (Default: `0.6`)
    - **What it does:** Determines the fraction of usable memory (Total Heap minus the 300MB Reserved Memory) dedicated to the Spark Memory pool (Storage + Execution). The remainder goes to User Memory.
    - **Tuning tip:**
        - _Decrease it_ if your UDFs or internal custom data structures create massive objects and cause OOM errors in User Memory.
        - _Increase it_ (e.g., to `0.8`) if your operations rely entirely on Spark DataFrames/SQL and you don't create many custom Java/Python objects, giving Spark more room to execute.
            
- **`spark.memory.storageFraction`** (Default: `0.5`)
    - **What it does:** Sets the "immunity boundary" within the Spark Memory pool. It defines the fraction of Spark Memory that is immune to eviction by Execution operations.
    - **Tuning tip:**
        - _Increase it_ (e.g., to `0.7`) if your application is iterative (like Machine Learning) and heavily relies on cached data.
        - _Decrease it_ (e.g., to `0.3` or lower) if you rarely cache data but perform massive aggregations, joins, or shuffles, which require high execution memory.
    
## **3. Concurrency & Task Memory**
Memory isn't just about the total size; it's about how many tasks are sharing it at the same time.
- **`spark.executor.cores`** (Default: `1` in YARN, all available in Standalone)
    - **What it does:** Determines the number of concurrent tasks an executor can run.
    - **Tuning tip:** Memory is shared among all active tasks in an executor. If `spark.executor.memory` is 8GB and you have 4 cores, each task effectively gets 2GB. If tasks are running out of memory, **decreasing the number of cores per executor** gives each remaining task a larger slice of the memory pie. A sweet spot is usually 4 to 6 cores per executor to maximize HDFS throughput without overwhelming the GC.

### **Common Tuning Scenarios**
- **Scenario A: High Garbage Collection (GC) Pauses**
    - _Solution:_ Switch to G1GC garbage collector (`spark.executor.extraJavaOptions=-XX:+UseG1GC`). Reduce `spark.memory.fraction` to give more space for GC to operate in User Memory.
- **Scenario B: PySpark OOM Errors**
    - _Solution:_ PySpark runs Python worker processes outside the JVM. Therefore, you must significantly increase `spark.executor.memoryOverhead` to accommodate the Python processes, or use `spark.memory.offHeap.enabled`.
