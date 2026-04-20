---
up:
  - "[[003_skills/data-engineering/04 processing layer|04 processing layer]]"
down:
prev:
topic: false
question: What is the use of resource manager and node manager?
---
# What is the use of resource manager and node manager?


> [!Summary] Summary
> 
> Resource manager
> 
> - It manages resource of entire Hadoop cluster.
> - It decides the resource allocation to each application
> - It schedule application using FIFO, capacity scheduler Fair scheduler
> - Tracks availability of resource in all node
> 
> Node manager
> - The node manager run on each node and manages node level resource
> - It manages container on its node
> - launches and monitors container
> - reports node status and health to resource manager
> - kills container if they exceeds allocated resource

## Resource Manager (RM)

### **Use / Role**

The **Resource Manager** is the **master daemon** responsible for **cluster‑wide resource management and scheduling**.

### **Main Functions**

*   Manages **CPU and memory resources** across the entire Hadoop cluster
*   Decides **which application gets how many resources**
*   Schedules applications using schedulers like:
    *   FIFO
    *   Capacity Scheduler
    *   Fair Scheduler
*   Tracks availability of resources on all nodes
*   Communicates with **Application Masters** and **Node Managers**

### **Key Point**

> Resource Manager **does not execute tasks** — it only **allocates resources**

***

## Node Manager (NM)

### **Use / Role**

The **Node Manager** is a **slave daemon** that runs on **each node** in the cluster and manages **node‑level resources**.

### **Main Functions**

*   Manages **containers** on its node
*   Launches and monitors containers for applications
*   Monitors:
    *   CPU usage
    *   Memory usage
    *   Disk
*   Reports node status and health to the ResourceManager
*   Kills containers if they exceed allocated resources

### **Key Point**

> NodeManager **executes and monitors tasks** on a specific node

***

## 🔁 How They Work Together

1.  A job is submitted to YARN
2.  **Resource Manager** allocates resources
3.  **Node Manager** launches containers on assigned nodes
4.  NodeManager monitors execution and reports back to ResourceManager

***

## 📊 Comparison Table

| Feature           | ResourceManager                  | NodeManager                      |
| ----------------- | -------------------------------- | -------------------------------- |
| Level             | Cluster‑wide                     | Node‑level                       |
| Type              | Master daemon                    | Slave daemon                     |
| Responsibility    | Resource allocation & scheduling | Container execution & monitoring |
| Runs on           | Master node                      | Every worker node                |
| Resource handling | Global view of resources         | Local resource usage             |
| Task execution    | ❌ No                             | ✅ Yes                            |
