---
up:
  - "[[003_skills/data-engineering/0602 Data abstraction|0602 Data abstraction]]"
down:
prev:
topic: false
question: What is task, Jobs and stages in Spark?
---
# What is task, Jobs and stages in Spark?
> [!Summary] Summary
> - When an action is trigger a job is created 
> 	- 1 action = 1 Job 
> - A job can have multiple stages, a stage is created when shuffle is required for an operation 
> 	- No of stages = No of wide transform + 1 
> - Each stage has multiple task, a task is associated with partitions of data
>
> | **Level** | **Scale**      | **Triggered By**                       | **Location**       |
> | --------- | -------------- | -------------------------------------- | ------------------ |
> | **Job**   | Global         | An Action (`count`, `save`, etc.)      | Driver             |
> | **Stage** | Group of Tasks | Shuffle boundaries (Wide dependencies) | Driver / Scheduler |
> | **Task**  | Unit of Work   | Number of partitions in a stage        | Executor           |
> 


## **1. Jobs (The "What")**
A **Job** is the highest level of the Spark execution hierarchy. It is triggered whenever you call an **Action** (like `collect()`, `saveAsTextFile()`, or `count()`) on an RDD or DataFrame.
- **Trigger:** One Action = One Job.
- **Scope:** If your Spark script has three `collect()` calls, you will see three Jobs in the Spark UI.

## **2. Stages (The "Where to Pause")**
Spark breaks a Job down into **Stages**. A new stage is created whenever there is a **Shuffle** (a Wide Dependency).
- **Narrow Dependencies:** Operations like `map()` or `filter()` don't require data to move between nodes. These are grouped into the same stage (Pipelining).
- **Wide Dependencies:** Operations like `reduceByKey()`, `join()`, or `groupBy()` require data to be reorganized across the cluster. This boundary forces Spark to finish all previous work before moving on, creating a new Stage.
    
## **3. Tasks (The "Doing")**
A **Task** is the smallest unit of work in Spark. It is a single operation performed on a single **partition** of data.
- **Parallelism:** If you have 10 partitions of data and a stage consists of a `map()` operation, Spark will launch 10 Tasks.
- **Execution:** Tasks are sent by the Driver to the Executors. One Task runs on one CPU core.

## **The Workflow in Action**
1. You write code with multiple transformations (`map`, `filter`, `join`).
2. You call `.collect()`. This creates a **Job**.
3. The **DAG Scheduler** looks at your transformations. It sees a `join` and splits the Job into **Stage 1** (before the join) and **Stage 2** (after the join).
4. If Stage 1 needs to process 100 blocks of data, Spark launches 100 **Tasks** to run across your executors simultaneously.
