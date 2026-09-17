---
up:
  - "[[003_DetailedNotes/Big data/Data bricks/Databricks_Fundamentals/Databricks_Fundamentals|Databricks_Fundamentals]]"
tags:
  - DataBricks/fundamentals
index: 4
type: topic
---
# Compute options

```ad-summary
collapse: true 
title: Summary

## Purpose

- Compute acts as the ==fundamental engine for all operations==, from interactive notebooks to complex Apache Spark pipelines. 
- The total cost is split into two categories: 
	- **Cloud Infrastructure Cost** for the underlying hardware (such as VMs on Azure or AWS) 
	- **Databricks Unit (DBU) Cost** for platform usage, monitoring, and security services.

## Core Compute Types

Different workloads require distinct cluster configurations for optimal performance and cost efficiency.

| Compute Type       | Primary Use Case                                 | Lifecycle & Cost Dynamics                                              |
| ------------------ | ------------------------------------------------ | ---------------------------------------------------------------------- |
| **All-Purpose**    | Interactive data exploration and development.    | Manually managed; highly flexible and fully customizable environments. |
| **Job Compute**    | Automated, scheduled data engineering workloads. | Auto-provisions and terminates upon completion; highly cost-optimized. |
| **Instance Pools** | Reducing cluster start times for auto-scaling.   | Maintains idle VMs for instant access; charges apply for idle runtime. |
| **SQL Warehouse**  | Executing BI queries and data warehousing tasks. | Tailored for SQL; available in Classic, Pro, and Serverless tiers.     |

## Serverless Architecture

Serverless compute abstracts away infrastructure management, allowing teams to focus on data processing rather than acting as cloud architects.

- Resources are **instantly assigned from a warm pool**, eliminating the standard 5–7 minute hardware allocation wait time.
- The architecture **removes the need to manually configure** VM types, memory ratios, or auto-scaling bounds.
- **Total Cost of Ownership is often lower by eliminating idle cluster time** and separate hardware billing.
- **Built-in guardrails**, such as Overspend Protection, automatically terminate inactive sessions to manage budgets effectively.
## Governance & Permissions

Administrators manage costs and standardize resources across teams by enforcing **Compute Policies** and setting precise access levels.
- P**olicies dictate configuration limits**, such as maximum node counts, mandatory termination limits, and approved VM types.
- **Updates to JSON policy definitions** do not auto-restart active clusters; administrators must **manually enforce compliance** on the next restart.
- Access controls range from basic notebook attachment ("Can Attach To") to full configuration and termination authority ("Can Manage").

```
---
# Compute Options

## Purpose of Compute in Databricks

- Compute is the engine that runs all operations in Databricks
- Including interactive notebook workloads, SQL queries, ETL/ELT pipelines, machine learning experiments, and ad-hoc data exploration. 
- Every operation requires a compute resource (cluster).

### Cost Components
The cost of Databricks compute is generally split into two parts:
* **Cloud Infrastructure Cost (e.g., Azure VM Cost):** Payment to the cloud provider for the underlying hardware. Depends on VM size, region, and type (Spot vs. On-Demand).
* **Databricks Unit (DBU) Cost:** Payment to Databricks for platform usage (Runtime, workspace services, security, monitoring). 
* *Typical Hardware Ratio:* Clusters commonly follow a 1 CPU : 4 GB RAM ratio.

## 1. All-Purpose Compute (Classic Compute)

This is a versatile, dedicated server environment typically used by power users and developers for exploratory data analysis, running notebooks, or working with SQL workbooks.
- **Architecture:** It utilizes a configuration where a single machine can host both the driver and the worker node, or it can be scaled across a cluster.
- **Flexibility:** Offers complete control over the runtime environment, allow listing custom configurations, libraries, and direct attachment to interactive notebooks.

### 🛠️ Steps to Create:
1. Navigate to the **Compute** section in the left sidebar and click **Create**.
2. Provide a **Unique Compute Name**.
3. Select a **Policy** depending on the restriction level needed:
    - _Unrestricted:_ All configuration options are available.
    - _Personal Compute:_ A subset of smaller-sized compute options meant for individual use.
    - _Power User / Shared Dedicated Resource / Legacy (Hive Metastore)_.
4. Choose the workload type (e.g., untick **Machine Learning** if standard data processing is needed).
5. Select the **Spark Version** (Databricks Runtime).
6. Toggle **Photon Acceleration** if higher performance is desired (Note: this consumes higher DBUs).
7. Select the node type: Choose between a **Single Node** or **Multi-Node** environment.
8. Set the **Worker Type (VM type)** and configure **Auto Scale** by defining the **Minimum and Maximum number of workers**.
9. Set the **Terminate After** duration (idle timeout) to prevent unnecessary costs.
10. Add **Tags** for billing and cost-tracking purposes.
11. Click **Create**.

## 2. Job Compute
Job compute is a cost-optimized resource engineered explicitly for automated, scheduled production workloads.
- **Lifecycle:** Unlike all-purpose compute, you cannot manually create a Job compute cluster directly to sit idle. It is spun up automatically when a scheduled job starts and is completely terminated and destroyed upon job completion.
- **Cost Efficiency:** Because it lacks the overhead of interactive tools and only runs for the duration of the task, it typically has a lower DBU cost structure.
- **Management Permissions:** Configured with permissions such as _Can Attach_ (allows notebook attachment), _Can Restart_ (start, restart, or terminate), and _Can Manage_ (edit compute details).
### 🛠️ Steps to Provision:
1. Open an existing **Notebook** or workflow.
2. Click on the scheduling option to **Schedule a Job**.
3. Define the job parameters, tasks, and scheduling intervals.
4. Under the compute option within the job details, specify the required configurations (similar options to all-purpose compute are available here).
5. Save the job. Databricks will handle the automated creation and teardown of this compute dynamically at runtime.

## 3. Instance Pools

Pools are a set of idle, ready-to-use virtual machine instances designed to radically reduce cluster start times and auto-scaling delays.
- **Mechanism:** Pools keep a minimum number of instances "always on" in an idle state. When a cluster requests a node, it pulls from the pool instantly instead of waiting 5–7 minutes for cloud provider hardware allocation.
- **Cost Trade-off:** While reducing boot times significantly, users are charged for the idle runtime of instances maintained within the pool.

### 🛠️ Steps to Create:
1. Navigate to the **Pools** UI within the compute interface and click **Create**.
2. Give the pool a distinct **Name**.
3. Set the **Min Idle** (the minimum number of instances kept warm and ready).
4. Set the **Max Capacity** (the maximum ceiling of instances the pool can grow to).
5. Configure the **Idle Instance Auto-Termination** window (in minutes) to determine when excess idle instances are spun down.
6. Choose the **Instance Type (VM Type)** and toggle **Photon Acceleration** if required.
7. Select the allocation type:
    - _On-Demand:_ Reliable, dedicated servers that stay active.
    - _Spot:_ Significantly cheaper shared resources, but vulnerable to being evicted at any time by the cloud provider.

## 4. SQL Warehouse
SQL Warehouses are highly tailored compute resources optimized purely for executing SQL queries, building BI dashboards, and data warehousing tasks.
- **Types:** Available in three tiers—**Classic**, **Pro** (optimized performance), and **Serverless**.
- **Performance:** Serverless options provide instant access, abstracting away underlying cloud infrastructure and booting faster than any traditional instance pool.

### 🛠️ Steps to Create:
1. Go to the **SQL Warehouses** section in Databricks.
2. Provide a **Name** for the warehouse.
3. Select the **Cluster Size** (ranging across T-shirt sizes like 2X-Small, Small, Medium, etc.).
4. Choose the **Type** of warehouse: **Classic**, **Pro**, or **Serverless**.
5. Configure the scaling bounds by adjusting **Scaling Min** (typically 1) and **Scaling Max** based on concurrency requirements.
6. Adjust the **Auto-Stop** threshold (e.g., set to 60 minutes or significantly lower for serverless to optimize spend).
7. Click **Create**.

## Serverless

- Serverless compute in Databricks is an architecture where Databricks completely manages the underlying cloud infrastructure (the servers/virtual machines) on your behalf. 
- Instead of you having to configure, provision, and maintain clusters in your own cloud account (like Azure, AWS, or GCP), Databricks handles it all in the background.

Here is a breakdown of how it works and why it’s considered the future of data processing:
### 1. How It Works
- **Decoupled Architecture:** 
	- In a traditional setup, you have to spin up a cluster in your cloud environment (which takes 5–7 minutes) before you can run a query. 
	- With serverless, Databricks maintains a warm pool of resources on their end.
- **Instant Access:** 
	- When you run a notebook, execute a job, or run a SQL query, the serverless compute automatically assigns resources to your task almost instantly (usually in under 15 seconds).
- **Version-less:** 
	- You don't have to worry about selecting or upgrading Databricks Runtime versions. 
	- The serverless engine is constantly updated and optimized by Databricks behind the scenes.

### 2. Key Benefits
- **Zero Infrastructure Management:** 
	- Your data engineering and data science teams don't need to act as cloud architects. 
	- They don't have to configure VM types, memory ratios, or auto-scaling bounds.
- **Cost Efficiency (Total Cost of Ownership):** 
	- While the per-DBU cost might look slightly higher on paper (e.g., $0.95/DBU vs. $0.85/DBU for classic), it often ends up being cheaper. 
	- You are not paying your cloud provider separately for the underlying hardware, and because it spins up and down instantly, you aren't paying for idle "warm-up" or "cool-down" time.
- **Built-in Guardrails:** 
	- It comes with features like Serverless Overspend Protection, which automatically stops interactive notebooks after a set period (like 2.5 hours) to prevent runaway costs, and budget policies to track user spending.
    
### 3. Primary Use Cases
- **Databricks SQL (Serverless SQL Warehouses):** This is the most common use case. BI tools (like Power BI or Tableau) need to query data instantly. Serverless SQL provides that immediate, low-latency response time without needing a cluster running 24/7.
- **Interactive Notebooks:** Data scientists can start writing code and exploring data immediately without waiting for a personal cluster to boot up.
- **Production Workflows:** Jobs can be scheduled to run on serverless compute, ensuring that resources are only consumed exactly when the job is running.

## Compute Policies

- In Databricks, policies are rule sets used to **restrict the compute configuration** options available to users when they create or modify a cluster. 
- They are essential for enforcing governance, controlling costs, and standardizing workloads.
- You can use default configurations or add your own custom policies to suit your organization's needs. 
- When creating a policy, you have the option to use a policy "family," which acts as a foundational template.

- Some of the common policy types and templates include:
	- **Unrestricted:** All compute configuration options are available to the user without limitations.
	- **Personal Compute:** A restricted subset of options specifically designed for small-size compute workloads.
	- **Power User**.
	- **Shared Dedicated Resource**.
	- **Legacy (Hive Metastore)**.
	- **Budget Policy (Serverless):** A specialized policy used in serverless environments to monitor and manage how many DBUs a user has consumed.

### How to Create a New Compute Policy
- Administrators can configure policies using the Databricks UI or by writing them programmatically. 
- If you need to overwrite specific rules or implement advanced configurations, you can update the policy definition directly in JSON format.
- Here are the standard steps to create a new compute policy:
	1. Navigate to the **Compute** section in the left-hand sidebar of your Databricks workspace.
	2. Select the **Policies** tab at the top of the page.
	3. Click the **Create policy** button.
	4. **Name the Policy:** Provide a clear, unique name (e.g., `Marketing_Team_Standard_Cluster`).
	5. **Select a Policy Family:** Choose a base template from the "Family" dropdown menu to inherit default rules.
	6. **Set Constraints:** Use the UI fields to define restrictions, such as maximum node counts, mandatory auto-termination limits, or approved virtual machine types.
	7. **JSON Configuration:** If you need more granular control, click **Edit definition as JSON** to manually write or overwrite the policy rules in JSON syntax.
	8. Assign required tags (for billing purposes) and specify any compute-scoped libraries that should be installed automatically.
	9. Click **Create**.

### Updating and Enforcing Policies
Organizational requirements change, meaning policies will often need to be modified after they are created.
- **Editing:** You can edit an existing policy at any time, typically by updating the JSON definition or adjusting the UI toggles.
- **Compliance:** 
	- When you update a policy, existing clusters do not automatically restart. 
	- You may need to manually update the active compute resources so that they comply with the newly updated policy. 
	- Databricks provides an "Enforce" mechanism within the policy UI to push these updates to out-of-compliance clusters on their next restart.

### Permissions
Define what level of access a user has on a specific cluster:
* **Can Attach To:** Can attach a notebook but cannot start/restart.
* **Can Restart:** Can start, restart, and terminate (includes 'Can Attach To').
* **Can Manage:** Highest access. Can edit settings, resize, modify permissions, and terminate.