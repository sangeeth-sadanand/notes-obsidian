---
up:
  - "[[003_skills/data-engineering/06 Spark/0603 SQL/060304 Optimization/060304 Optimization|060304 Optimization]]"
down:
prev:
topic: false
question: Give an overview of spark memory split?
---
# Give an overview of spark memory split?


> [!Summary] Summary
> - Spark memory has **On-heap** managed by Spark JVM and **off-heap** memory. 
> - On heap memory is broadly split into 3 part controlled by **spark.executor.memory** 
> 	1. Overhead memory - 3001913 / 10% of executor memory 
>     2. User memory - Used for user data structure, UDF and internal metadata (40% default) 
>     3. Spark memory - 160% default) controlled by Spark. memory. fraction. It is divided into Execution memory and storage memory 
> 			- Execution memory is used for shuffles, joins and sorts 
> 			- Storage memory is used for cached RPD and broadcast 
> 			- It is a dynamic boundary if the process needs more execution memory it evicts storage memory 
> 			- we can control using spark. memory. storage Fraction to increase storage memory. 
> - Off - heap memory controlled by Spark - memory • off Heap - size. It can be used for execution & Storage memory.
> ![[001_Meta/media/spark_memory_management.svg|640]]

Apache Spark manages memory primarily through its **Unified Memory Management** architecture (introduced in Spark 1.6). This model dynamically shifts memory between storage (caching) and execution (computations), ensuring that neither subsystem starves the other while maximizing resource utilization.
Here is an overview of how Spark splits and manages its memory, followed by the specific configurations and an SVG diagram.

## **The Spark Memory Split (On-Heap)**
When you allocate memory to a Spark executor, it is divided into three primary regions:
- **Reserved Memory:** Spark hardcodes a small chunk of memory (typically 300 MB) completely out of the user's control. This ensures that the engine has enough memory to recover from OutOfMemory (OOM) errors and perform internal system operations.
- **User Memory:** This region stores user-defined data structures, internal metadata, and objects created by User Defined Functions (UDFs). It is the memory left over after Spark reserves its own slice. Spark does not monitor this space, so if your UDFs create massive objects, you can still encounter an OOM error here.
- **Spark Memory:** This is the core memory pool managed directly by Apache Spark, and it is divided into two sub-regions separated by a dynamic boundary:
    - **Storage Memory:** Used for caching RDDs, DataFrames, and storing broadcast variables.
    - **Execution Memory:** Used for transient data during computations like shuffles, joins, sorts, and aggregations.
    - _Dynamic Boundary:_ If Execution needs more space and Storage is not using its full allocation, Execution can borrow it (and vice versa). However, Execution memory can evict Storage memory blocks if necessary, but Storage cannot evict active Execution blocks.

## **Memory Configurations: Heap vs. Off-Heap**
Spark allows you to utilize both the standard JVM Heap memory and Off-Heap memory (direct RAM outside the JVM).
### **1. Heap Memory Configurations**
This is the standard memory inside the Java Virtual Machine. It is subject to JVM Garbage Collection (GC), which can introduce performance pauses if not tuned correctly.
- `spark.executor.memory`: The total amount of memory requested for the JVM heap (e.g., `4g`, `8g`).
- `spark.memory.fraction`: Determines the percentage of usable memory (Total - Reserved) allocated to **Spark Memory**. The default is **0.6** (60%). The remaining 40% becomes User Memory.
- `spark.memory.storageFraction`: Determines the baseline split between Storage and Execution within the Spark Memory pool. The default is **0.5** (50% to Storage, 50% to Execution).

### **2. Off-Heap Memory Configurations**

Off-heap memory allows Spark to bypass the JVM entirely. This is highly beneficial for large datasets because it completely eliminates Garbage Collection overhead and reduces memory footprint (since objects are serialized).
- `spark.memory.offHeap.enabled`: Must be set to `true` to use off-heap memory (default is `false`).
- `spark.memory.offHeap.size`: The absolute amount of memory to allocate off-heap (e.g., `5g`).

>[!Tip]
> Off-heap memory does not have "User Memory" or "Reserved Memory" splits. The entire allocated size is dedicated purely to the Spark Memory pool (Storage and Execution).


