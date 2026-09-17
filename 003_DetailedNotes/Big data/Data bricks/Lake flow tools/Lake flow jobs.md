---
up:
  - "[[003_DetailedNotes/Big data/Data bricks/Lake flow tools/Lake flow tools|Lake flow tools]]"
tags:
  - DataBricks/LakeFlow/jobs
index: 3
type: topic
---
## Lake flow jobs

```ad-summary
collapse: true 
title: Summary

## What is Lakeflow Jobs?

**Lakeflow Jobs** is Databricks' native workflow orchestration service within the Lakeflow platform.

Lakeflow consists of:

- **Lakeflow Connect** → Data ingestion
- **Lakeflow Declarative Pipelines (DLT)** → Data transformation (ETL)
- **Lakeflow Jobs** → Workflow orchestration

Its role is to coordinate and automate ingestion, transformation, analytics, and other workloads across the Databricks ecosystem.

## Why Use Lakeflow Jobs Instead of External Orchestrators?

Traditional orchestrators:

- Apache Airflow
- Azure Data Factory (ADF)

Challenges:

- Limited awareness of Delta Lake and DLT pipelines
- Additional integration and maintenance effort
- Fragmented monitoring and governance

**Lakeflow Jobs provides native Databricks orchestration**, giving tighter integration, better observability, and simplified management.


# Core Building Blocks

## Job

A **Job** is a complete workflow containing one or more tasks.

Responsibilities:

- Execution management
- Scheduling
- Monitoring
- Error handling
- Dependency management

## Task

A **Task** is the smallest executable unit within a Job.

Characteristics:

- Executes one workload
- Independently configurable
- Supports multiple workload types

Supported task types:

- Notebook
- Python script
- SQL query/file
- Lakeflow Declarative Pipeline (DLT)

# Task Execution Patterns

Lakeflow Jobs supports:
### Sequential

Bronze → Silver → Gold
### Fan-Out
~~~
Task1  
├── Task2  
└── Task3

~~~

### Funnel
~~~
Task 2
	\
	 --> Task 1
	/
Task 3
~~~

### Conditional Execution
Run tasks only when conditions are met.

### For-Each Execution
Iterate over a list of values and execute nested tasks for each item.


## Parameters vs Task Values

### 1. Job Parameters
- Defined once at job level.
- Purpose:
	- Shared configuration across all tasks
	- Environment management
	- Avoid hardcoding

- Examples:
	catalog_name = training_catalog  
	environment = prod

- Use when:
	- Same value required by all tasks

### 2. Task Parameters

- Defined at individual task level.
- Purpose:
	- Task-specific inputs
	- Notebook/script reuse

- Example:
	Task1 → bronze_schema  
	Task2 → silver_schema  
	Task3 → gold_schema

Use when:
	- Different tasks need different values

#### Parameter Priority
- If both exist with the same key:
- Job Parameter > Task Parameter
- Job parameter overrides task parameter.

### Task Values

- Task Values enable **runtime communication between tasks**.
- Unlike parameters:
	- Parameters = known before execution
	- Task Values = created during execution
- Example:
	- Upstream Task
	~~~python
dbutils.jobs.taskValues.set(  
key="has_duplicates",  
value=True  
)
	~~~

	- Downstream Task
	~~~python
dbutils.jobs.taskValues.get(
taskKey="orders_silver_cleaned",
key="has_duplicates"
)
	~~~

- Common uses:
	- Passing runtime outputs
	- Conditional workflows
	- Sharing computed results


## Dynamic Value References

- Use `{{ }}` syntax to reference runtime information.
- Examples:
	- Job metadata:
	~~~python
{{job.start_time.day}}
	~~~
	- Task metadata:
	~~~python
	{{task.name}}
	~~~

	- Task values:
	~~~python
	{{tasks.orders_silver_cleaned.values.has_duplicates}}
	~~~


- Used in:
	- Job settings
	- Task configuration
	- YAML definitions

## Feature Comparison

|Feature|Purpose|
|---|---|
|Job Parameters|Global configuration|
|Task Parameters|Task-specific configuration|
|Task Values|Runtime communication|
|Dynamic Value References|Access runtime metadata and outputs|

## Compute Options

### Interactive Cluster

- Shared cluster
- Development/testing
- Reusable across jobs

### Job Cluster

- Created per execution
- Auto-terminated
- Recommended for production

### Serverless Compute

- Fully managed
- Fast startup
- No cluster management

### SQL Warehouse

- Dedicated SQL execution engine
- BI and analytics workloads



## Compute Best Practice

- Production:
	Job Cluster  
	OR  
	Serverless Compute

Avoid using Interactive Clusters for critical production workloads.


## Serverless Performance Modes

### Standard (Performance Optimised = Off)

- Lower cost
- Longer startup time
- Suitable for non-urgent batch jobs

### Performance Optimised = On

- Faster startup
- Better SLA compliance
- Higher cost

Suitable for:

- Production pipelines
- Time-sensitive workloads



## Common Task Configurations

### Task Parameters

Provide task-specific settings.
### Dynamic Value References

Access job metadata and upstream outputs.

### Notifications

Alert on:

- Success
- Failure
- Duration thresholds

### Retry Policies

Automatically retry failures based on configured rules.

## Trigger Types

### Scheduled Trigger

Time-based execution.

Examples:
	Every hour  
	Daily  
	Cron schedules

### Continuous Trigger

Run → Complete → Restart

Used for near real-time workloads.

### File Arrival Trigger

Starts when new files arrive in:
- Cloud storage
- Volumes
- Landing zones

### Table Update Trigger

Runs automatically when a table changes.

### Manual Trigger

Started on demand for:
- Testing
- Debugging
- Validation

## For-Each Pattern

Structure:
~~~
For-Each Container Task  
		↓  
	Nested Task
~~~

Common use cases:

- State-wise processing
- Country-wise processing
- Monthly partition processing
- Historical backfills


## Best Practices

✅ Use **Job Clusters** or **Serverless** in production.
✅ Use **Service Principals** as job owners instead of personal accounts.
✅ Use **Repair & Rerun** to rerun only failed tasks.
✅ Parameterise jobs and tasks rather than hardcoding values.
✅ Use Task Values for runtime communication.
✅ Use Dynamic Value References for metadata-driven workflows.
```
---
* Lakeflow provides a unified platform covering the full data lifecycle: 
	* Lakeflow Connect for data ingestion,
	* Lakeflow Declarative Pipelines for data transformation (ETL), and 
	* Lakeflow Jobs for workflow orchestration.

* Lakeflow Jobs is responsible for orchestrating and coordinating workloads across ingestion and transformation layers.

### Traditional vs. Native Orchestration

* Traditionally, orchestration has been handled using third-party tools such as Apache Airflow or Azure Data Factory (ADF).
* **Challenges with Third-Party Tools:**
	* They are not natively unified with Delta Lake.
	* They have limited awareness of 
		* Delta tables, 
		* pipeline states
		* Databricks-native workloads.
	* Integration and maintenance overhead increases.
	* Governance and observability become fragmented.
* Lakeflow Jobs addresses these challenges by providing native orchestration tightly integrated with the Databricks ecosystem.

### Core Concepts: Jobs and Tasks

* **What Is a Job?:** A Lakeflow Job represents a complete workflow and acts as a container for execution logic.
* **Tasks:**
	* A job consists of one or more tasks.
	* The job defines how tasks are executed, configured, and monitored while coordinating the execution of ingestion, transformation, and other workloads.
	* A task is the smallest unit of work within a job.

* **Characteristics of a Task:**
	* Executes a single workload.
	* Can run different types of compute logic.
	* Is independently configurable within a job.

* Examples include ingesting customer data from a raw customer table or orders data from a cloud storage folder.

### Task Configuration & Capabilities

* **Supported Task Types:** 
	* Lakeflow Jobs support multiple task types, 
		* including notebooks, 
		* declarative pipelines (DLT), 
		* SQL queries or SQL files, and 
		* Python scripts. 
	* This flexibility allows orchestration of both data engineering and analytics workloads.

* **Task-Level Configuration:** 
	* When defining a job, each task can be independently configured, including paths to code (notebook, script, or SQL file), runtime parameters, notification settings, and retry policies and failure handling. 
	* This enables fine-grained control over execution behavior.

* **Control Flow Between Tasks:** 
	* Lakeflow Jobs allow defining execution relationships between tasks, 
		* supporting sequential execution, 
		* parallel execution, 
		* conditional execution, and 
		* for-each (iterative) execution to construct complex workflows within a single job.


    
### Job Parameters

* A Job Parameter is a key-value pair defined at the job level, and it is automatically available to all tasks inside that job.
* **Why We Use Job Parameters**:
	* Avoid hardcoding values.
	* Maintain consistency across all tasks.
	* Easily switch environments (Dev → QA → Prod).
	* Reuse the same workflow in multiple contexts.
* **How It Works**: 
	* You define a parameter in the job configuration
		* (e.g., Key: `catalog_name`, Value: `training_centralindia_lakeflowjobs_dbws`). 
	* This value is pushed to all tasks in that job.

### Task Parameters

* A Task Parameter is a key-value pair defined at the individual task level, not the whole job.
* **Why We Use Task Parameters**:
	* When different tasks need different inputs.
	* When tasks perform different operations.
	* When you want flexibility per task.

* **Example Scenario**: 
	* Imagine a job with 3 tasks: 
		* Task 1 (Load Bronze Layer), 
		* Task 2 (Transform to Silver), and 
		* Task 3 (Create Gold Aggregation). 
	* If each task needs a different schema (`bronze_schema`, `silver_schema`, `gold_schema`), instead of creating separate notebooks, you pass different task parameters, use the same notebook, and dynamically switch logic.

* **Notebook Implementation**: 
	* `dbutils.widgets.text("schema_name", "")
	* `schema_value = dbutils.widgets.get("schema_name"))`.


* **When to Use What?**:
	* Use Job Parameters when the value is common across all tasks, for environment-based configs, or for catalog/database names.
	* Use Task Parameters when each task needs a different input, the same notebook is reused for different purposes, or specific runtime configuration per task is required.

> [!Note]
> If a Job Parameter and a Task Parameter have the same key name, the Job Parameter takes precedence (higher priority) and overrides the task-level value.

### Task Values & Dynamic Value References

* **Task Values** allow one task to set a value during execution so that another task can retrieve it dynamically.
* They are used for passing runtime results between tasks, conditional logic, and sharing computed outputs.
* Parameters are defined before execution, while Task Values are created during execution, making task values more dynamic and advanced.
* **Why We Need Task Values?**: 
	* For instance, if Task 1 checks if duplicates exist in data, 
	* Task 2 can behave differently based on that result determined at runtime.

* **Syntax Examples**:
* Setting a Task Value (Upstream Task):
	* `dbutils.jobs.taskValues.set(key="has_duplicates", value=duplicate_exists)`.
* Getting a Task Value (Downstream Task):
	* `dbutils.jobs.taskValues.get(taskKey="orders_silver_cleaned", key="has_duplicates")`.

* **Example Flow (Task1 → Task2)**: Task 1 sets `catalog_name` to `databrickslearning`, and Task 2 retrieves it dynamically.

* **Dynamic Value References** allow you to reference job metadata, task metadata, task outputs, and runtime values using `{{ }}` syntax.
	* They are mostly used inside job configuration, task configuration, YAML, and UI job settings.
	* Examples include Job Context (`{{job.start_time.day}}`), Task metadata (`{{task.name}}`), and Accessing Task Values (`{{tasks.orders_silver_cleaned.values.has_duplicates}}`).

### Parameter and Value Comparison

| Feature                     | When Used                        |
| --------------------------- | -------------------------------- |
| **Job Parameters**          | Global config for all tasks      |
| **Task Parameters**         | Task-specific config             |
| **Task Values**             | Runtime communication            |
| **Dynamic Value Reference** | Referencing runtime/job metadata |

### Accessing Upstream Outputs

We can access upstream outputs using:
* Job Parameters
* Task Parameters
* Task Values
* Dynamic Value References

Each serves a different purpose depending on:

* Static vs. runtime requirement
* Global vs. task-level scope
* Configuration vs. execution-level logic

### Compute Options in Lakeflow Jobs

* Each task within a Lakeflow Job can run on a specific compute configuration, sharing compute resources or using independent compute environments.
* **Types of Compute**:
	* **Interactive Cluster**: Shared cluster typically used for development and ad hoc analysis, reusable across multiple jobs, suitable for experimentation, but not ideal for isolated production workloads.
	* **Job Cluster**: Created specifically for a job run, automatically terminated after execution, ensures workload isolation, and recommended for production pipelines.
	* **Serverless Compute**: Fully managed by Databricks with no cluster management required, optimized startup and scaling, and suitable for automated job execution.
	* **SQL Warehouse**: Dedicated compute for SQL workloads used when executing SQL queries or dashboards, optimized for BI and analytics use cases.

* **Compute Assignment Strategy**: 
	* Tasks within the same job can 
		* share compute for cost efficiency
		* use different compute configurations for workload isolation (e.g., Bronze on Job cluster, Silver on Serverless, Gold on SQL Warehouse).

* **Performance Optimization Setting (Serverless Compute)**:
	* Performance Optimized = Off: 
		* Focuses on cost efficiency, 
		* longer startup times (4-6 minutes), 
		* \suitable for non-urgent workloads and 
		* batch pipelines with relaxed SLAs.
	* Performance Optimized = On: 
		* Faster startup and execution, 
		* designed for time-sensitive workloads, 
		* higher cost compared to standard mode, and 
		* ideal for production jobs with strict SLAs.

### Common Workload Patterns

* Lakeflow Jobs support multiple orchestration patterns.
	* **Sequential Pattern**: Bronze → Silver → Gold, where each task depends on the previous task, representing the most common Medallion architecture implementation and ensuring ordered execution.
	* **Funnel Pattern**: Multiple upstream sources (Task 2, Task 3) converging into a single downstream task (Task 4), used when multiple data sources need to be consolidated.
	* **Fan-Out Pattern**: A single upstream task (Task 1) triggering multiple downstream tasks (Task 2, Task 3), used when a dataset needs to be distributed across multiple processing paths.
### Common Task Configuration Options

* Each task can be configured independently.
	* **Task Parameters**: Pass configuration values specific to that task, used for schema names, flags, and environment-specific settings.
	* **Dynamic Value References**: Reference runtime values using `{{ }}` syntax to access job metadata or upstream task outputs.
	* **Notification Alerts**: Configure alerts for success, failure, and duration thresholds for operational monitoring and production readiness.
	* **Retry Configuration**: Jobs can automatically retry failed tasks based on defined policies considering failure types (transient vs. data quality), resource impact (cluster contention), and downstream dependencies (SLA compliance).

### Triggers in Lakeflow Jobs

* A trigger is a rule that automatically starts a job run based on a defined condition or schedule, enabling automation.
* **Types of Triggers**:
	* **Time-Based Schedule (Scheduled Trigger)**: Jobs run at defined time intervals supporting simple intervals and cron expressions for advanced scheduling.
	* **Continuous Trigger (Always On)**: Starts a new job run immediately after the previous run completes, used for fraud detection and near real-time data pipelines.
	* **File Arrival Trigger**: Automatically triggers a job when new files are detected in cloud storage paths, volume locations, or data landing zones.
	* **Manual Trigger**: Jobs executed on demand by a user for ad hoc runs, debugging, testing, and validation.
	* **Table Update Trigger**: Automatically starts a job when a specified table is updated, ensuring transformations occur immediately after upstream changes.

* **Scheduled Triggers - Design Considerations**: 
	* Ensure time zone alignment, 
	* avoid overlapping runs, 
	* consider data availability timing, and 
	* validate upstream dependency completion.

### For-Each Task Pattern

* Lakeflow Jobs support iterative execution using a For-Each pattern.
	* **Structure**: Consists of a Top-Level Container Task (manages the loop and iterates over input values) and a Nested Task (executes once per iteration and performs the actual processing logic).
* **Use Cases**:
	* Regional Processing: Process data independently for each state, region, or country.
	* Time-Based Partition Processing: Process data across monthly partitions, daily partitions, or historical backfills.
### Best Practices for Lakeflow Jobs

* Use Job Clusters or Serverless in Production to avoid interactive clusters, ensure workload isolation, and improve reliability.
* Use Service Principals for Job Ownership to avoid personal user dependency, improve security, and support automated deployments.
* Use Repair and Rerun Instead of Full Reprocessing to rerun only failed tasks, reduce compute costs, and improve operational efficiency.
* Parameterize Tasks to avoid hardcoding values, improve reusability, enable environment-based execution, and support CI/CD workflows.
