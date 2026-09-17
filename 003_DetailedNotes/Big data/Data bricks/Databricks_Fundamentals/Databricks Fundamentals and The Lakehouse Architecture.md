---
up:
  - "[[003_DetailedNotes/Big data/Data bricks/Databricks_Fundamentals/Databricks_Fundamentals|Databricks_Fundamentals]]"
tags:
  - DataBricks/fundamentals
index: 1
type: Group of topic
---
# Databricks Fundamentals & The Lakehouse Architecture

```ad-summary
collapse: true 
title: Summary

## What is Databricks

- A **platform** that provides simplified tools for Data engineering, Data science and AI
- Build on top of **Apache Spark** 

## Why
- Unifies **multiple tools** under one platform
- **Fully managed and auto-scaled** 
- Uses performant **spark engine** under the hood
- Reliable data lake using **delta lake**
- Uses **Lakehouse** architecture 

## Data lakehouse concept 

> [!Note]
> Data lakehouse = Data lake + Data warehouse

### Data lake
- Scalable
- Cost-effective 


### Data warehouse
- Performant
- Reliable

### Data lakehouse 

- Cost effective and scalable using object storage of S3, ADLS gen2 GCS
- **Deltalake tables** - Reliable and performant

## Modern Data Lakehouse Architecture

### Storage layer
Cloud providers like GCP, AWS and Azure provide scalable object storage

### Delta lake

File format that enables **ACID transactions**, **Schema enforcement**, **versioning and time travel**

### Unity Catalog

- Provide centralized **governance**, fine-grade access control, metadata management and data lineage tracking and audit compliance

## High level architecture 

### Control plane
- Controlled by **Databricks**
- It stores
	- Meta data
	- Access control list
	- Configs
	- Web UI

### Data plane
- Stored on **customer cloud account**
- This is used for:
	- Data storage
	- Classic compute
	- Networking component

## Eco-system 

- **Lakeflow connect** (ingestion)
- **Lakeflow declarative pipeline**(dependency management, monitoring tools)
- **Lakeflow Jobs** (Orchestration)
- **Unity catalog** (Centralized governance)
- **Delta lake** (Storage layer)

## Cost structure

>[!note]
>$$ 
>Total\ Cost = Azure\ VM\ Cost + Databricks\ DBU\ Cost
>$$

### Infrastructure Cost 
- It goes to the cloud provider
- You pay for the size of VMs, number of nodes and hours running

### Software Cost (Databricks Cost):
- Databricks charges separately in DBU (Databricks Unit).
- DBU depends on cluster type (All-purpose, Job, or Serverless) and runtime (Standard, Premium, Pro).
  

```

## What is Databricks?

- It is a simplified, unified platform for data engineering, data science and machine learning workflows.
- It uses and is built on top of Apache spark.

## Why Databricks?

| Traditional Tools                             | Databricks Solution                                                                              |
| :-------------------------------------------- | :----------------------------------------------------------------------------------------------- |
| **Multiple tools**                            | A unified ecosystem that includes ingestion, ETL, orchestration, and governance in one platform. |
| **Manual infrastructure management**          | Fully managed and auto-scaled clusters for simplified maintenance.                               |
| **Slow data processing**                      | Spark engine provides distributed and high-speed data processing.                                |
| **Lack of team collaboration**                | Collaborative notebooks supporting Python, SQL, R, and Scala within a shared workspace.          |
| **Unreliable data lakes**                     | Delta lakes include ACID transactions, schema enforcement, version control, and time travel.     |
| **Duplication between data lake & warehouse** | The lakehouse architecture merges both into a single system, eliminating redundant ETL.          |

## Data Lakehouse Concept

> [!Note]
>
> **Data Lakehouse = Data Lake + Data Warehouse**.


- **Data Lakes** provide scalable and cost-effective storage for large volumes of raw data but lack features such as ACID transactions and schema enforcement.
- **Data Warehouses** are reliable and more performant but are expensive and less flexible to handle unstructured data.
- Databricks creates a wrapper called **Delta Lake** on top of existing data lakes (Amazon S3, Azure Data Lake Storage Gen2, Google Cloud Storage).
- Delta Lake enables ACID transactions, schema enforcement, versioning, and time travel capabilities directly on data lake storage.

## Modern Data Lakehouse Architecture

1. **Storage Layer:** Cloud providers like GCP, AWS, and Azure provide scalable object storage, where all processed and raw data resides.
2. **Delta Lake:** This layer enhances basic cloud storage by adding _ACID transactions, schema enforcement, data versioning and time travel_, and optimized performance for large-scale processing.
3. **Unity Catalog:** Provides centralized governance, fine-grained access control, metadata management, data lineage tracking, and audit compliance.

> [!Note]
>
> This architecture combines the flexibility of data lakes and the reliability of warehouses.


# System Architecture & Ecosystem

## High-Level Architecture

Databricks operates in two planes:

### 1. Control Plane

This is the portion of Databricks fully controlled and hosted by Databricks. It stores:

- **Metadata:** Workspace metadata, table metadata, notebook and job metadata.
- **ACL (Access Control List):** User roles and permissions.
- **Configurations:** Cluster configuration templates, notebook settings, job information.
- **Web Application/UI**

### 2. Data Plane

This is where the actual data and compute live. It operates in the customer cloud account. The following are stored in the data plane:

- **User Data:** Files in ADLS/S3/GCS, Delta tables, DBFS.
- **Classic Compute Cluster:** Driver/worker nodes and runtime execution.
- **Networking Components:** VNETs, Subnets, Private endpoints.

|Control Plane|Data/Compute Plane|
|:--|:--|
|Workspace / Web App|Driver & worker execution|
|Notebook metastore|Networking (Spark)|
|Cluster & Job configs|Delta lake storage|
|ACL & User permission||
|Management & monitoring||

## Databricks Ecosystem

Databricks provides an integrated ecosystem that addresses every stage of the modern data lifecycle:

1. **Lakeflow Connect:** Handles data ingestion by enabling easy connectivity to a wide variety of data sources.
2. **Lakeflow Declarative Pipeline:** Automatically manages dependencies, monitors data quality, and handles incremental updates.
3. **Lakeflow Jobs:** Orchestrates, schedules, manages, and automates data workflows.
4. **Unity Catalog:** Provides a centralized solution for access, lineage, and compliance.
5. **Delta Lake:** Serves as the foundational storage and processing layer.

# Infrastructure, Compute, and Cost Management

## Backend Azure Resources

When a Databricks workspace is created, it creates its own managed resource group for backend resources such as:

- Worker VM
- Driver VM
- Network components
- Storage for cluster logs

## Compute Options in Databricks

1. **Standard or Classic Compute:**
    - Users manage clusters.
    - Users choose the number of nodes, node type, and runtime version.
    - Backend Azure VMs run as driver and worker.
2. **Serverless Compute:**
    - No VM provisioning, starts instantly, optimized auto-scaling.
    - Ideal for ad-hoc queries, notebooks, and lightweight Spark sessions.
    - Databricks manages the entire infrastructure.

## Databricks Cost Structure

Cost is determined by two components:

1. **Infrastructure Cost (Azure Cost):**
    - Databricks clusters run on Azure Compute (VM).
    - If you create a 3-node cluster, you are actually running 3 Azure VMs.
    - You pay for the size of VMs, number of nodes, and hours running.
2. **Software Cost (Databricks Cost):**
    - Databricks charges separately in DBU (Databricks Unit).
    - DBU depends on cluster type (All-purpose, Job, or Serverless) and runtime (Standard, Premium, Pro).

$$ 
Total\ Cost = Azure\ VM\ Cost + Databricks\ DBU\ Cost
$$