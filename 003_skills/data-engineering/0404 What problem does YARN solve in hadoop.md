---
up:
  - "[[003_skills/data-engineering/04 processing layer|04 processing layer]]"
down:
prev:
topic: false
question: What problem does YARN solve in hadoop?
---
# What problem does YARN solve in hadoop?


> [!Summary] Summary
> 
Problems before YARN (hadoop 1.x)
> 
> 1. Job tracker use to handle a lot of task from
> 2. Job scheduling
> 3. Resource management
> 4. Task monitoring
> 5. Fault tolerance
> 
> These caused scalability issue, Single point of failure, inefficient resource utilization, limited to map reduce task only.
> 
> YARN solved the issue by splitting the responsibility to resource manager, node manager and application manager.
> - Resource manager - manages cluster wide resources
> - Node manager - manages node level resource
> - Application manager - manages individual application
> 
> YARN provides dynamic allocation of resource, eliminates single point of failure and also supports multiple frameworks

### The Core Problem:

In **Hadoop 1.x**, the **Job Tracker** was responsible for **too many tasks**, which caused major limitations.

## Problems Before YARN (Hadoop 1.x)

In Hadoop 1.x, the **Job Tracker** handled:

1.  **Job scheduling**
2.  **Resource management**
3.  **Task monitoring**
4.  **Fault tolerance**

This design caused several issues:

### 1. Scalability Issue

*   One Job Tracker managed the entire cluster
*   As cluster size increased, Job Tracker became a **bottleneck**
*   Limited number of nodes and jobs could be supported

### 2. Single Point of Failure

*   If Job Tracker failed, **all running jobs failed**
*   Entire cluster processing stopped

### 3. Inefficient Resource Utilization

*   Resources were fixed as **map slots and reduce slots**
*   Idle map slots couldn’t be used for reduce tasks and vice versa

### 4. Limited to MapReduce Only

*   Hadoop could not easily support other processing frameworks like:
    *   Spark
    *   Tez
    *   Storm

## How YARN Solves These Problems

YARN **separates resource management from data processing**, making Hadoop more powerful and flexible.

## Key Solutions Provided by YARN

### 1. Separates Responsibilities

YARN splits Job Tracker functionality into:

*   **Resource Manager**
    *   Manages cluster-wide resources
*   **Application Master**
    *   Manages individual applications
*   **Node Manager**
    *   Manages resources on each node

 Result: **Better scalability and performance**

***

### 2. Dynamically Allocates Resources

*   Uses **containers** instead of fixed slots
*   Allocates CPU and memory as needed

 Result: **Efficient resource utilization**

***

### 3. Eliminates Single Point of Failure

*   Failure of one ApplicationMaster affects **only that application**
*   ResourceManager can be configured for high availability

 Result: **Improved fault tolerance**

***

### 4. Supports Multiple Processing Frameworks

YARN allows Hadoop to run:

*   MapReduce
*   Spark
*   Hive
*   HBase
*   Flink

 Result: Hadoop becomes a **general-purpose data processing platform**

***

### 5. Improves Cluster Utilization

*   Multiple applications share the same cluster resources
*   Better scheduling (FIFO, Capacity, Fair Scheduler)

 Result: **Cost-effective and optimized clusters**

***

## Summary: Problem vs Solution

| Problem in Hadoop 1.x   | How YARN Solves It                  |
| ----------------------- | ----------------------------------- |
| JobTracker bottleneck   | Distributed management              |
| Single point of failure | Application-level failure isolation |
| Fixed map/reduce slots  | Dynamic containers                  |
| MapReduce-only          | Multiple frameworks                 |
| Poor scalability        | Large cluster support               |

