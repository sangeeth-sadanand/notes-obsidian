---
up:
  - "[[000_+/0601 Architecture|0601 Architecture]]"
down:
prev:
topic: false
question: What is role of driver, cluster manager and executor manager?
---

# What is role of driver, cluster manager and executor manager?


> [!Summary] Summary
> Contents

In the Spark ecosystem, these three components work together in a hierarchical relationship to turn your code into distributed physical tasks.

---

## 1. The Driver (The Brain)

The Driver is the process where the `main()` method of your application runs. It is the "command center" that stays active for the entire duration of the application.

- **Analysis:** It converts your high-level code (SQL, Python, Scala) into a logical plan and then a physical plan called a **DAG** (Directed Acyclic Graph).
    
- **Scheduling:** It breaks the DAG into stages and tasks. It then schedules these tasks to be run on the Executors.
    
- **Coordination:** It tracks the status of all running tasks. If a task fails, the Driver is responsible for re-scheduling it.
    
- **Data Collection:** When you call an action like `.collect()` or `.take()`, the results are sent from the executors back to the Driver.
    

---

## 2. The Cluster Manager (The Matchmaker)

The Cluster Manager is an external service (not unique to Spark) that manages the physical resources of the cluster. Spark supports several, including **YARN**, **Kubernetes**, and **Mesos**.
  
- **Resource Allocation:** Its only job is to look at the available "inventory" of the cluster (CPU and RAM) and lease those resources to the Driver.
    
- **Isolation:** It ensures that different Spark applications (or other frameworks) running on the same cluster don't interfere with each other's memory or processing power.
    
- **The Hand-off:** Once the Cluster Manager allocates a "Worker Node" to your application, its involvement in the actual data processing is minimal; it simply monitors that the node stays alive.
    

---

## 3. The Executor (The Muscle)

The Executor is a distributed process that lives on a **Worker Node**. This is where the actual computation happens.

- **Task Execution:** It receives the serialized code (tasks) from the Driver and runs them on its local data partitions.
    
- **Storage:** It provides in-memory storage for RDDs or DataFrames that are **cached** or **persisted** by the user.
    
- **Reporting:** It sends the results and heartbeat signals (to prove it's still "alive") back to the Driver.
    

> **Note on "Executor Manager":** While we often talk about Executors, the term **"Executor Launcher"** or the **"Worker"** process acts as the manager on each node. It is responsible for starting and stopping the Executor processes as commanded by the Cluster Manager.

---

## The Workflow: How They Interact

|**Step**|**Action**|**Responsible Component**|
|---|---|---|
|**1**|User submits code; a session is created.|**Driver**|
|**2**|Application asks for 10 CPUs and 20GB RAM.|**Driver** $\to$ **Cluster Manager**|
|**3**|Resources found; processes started on nodes.|**Cluster Manager** $\to$ **Executors**|
|**4**|Tasks are sent to the nodes.|**Driver** $\to$ **Executors**|
|**5**|Data is processed; heartbeats sent back.|**Executors** $\to$ **Driver**|
