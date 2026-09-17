---
up:
  - "[[003_DetailedNotes/Big data/Data bricks/Lake flow tools/Lake flow tools|Lake flow tools]]"
tags:
  - DataBricks/LakeFlow/overview
index: 1
type: topic
---
# Overview

```ad-summary
collapse: true 
title: Summary

The Databricks platform unifies data and AI pipelines through a cohesive architecture built on top of Apache Spark.

**Core Platform Pillars**

* **Apache Spark:** The underlying distributed engine handling all batch, streaming, SQL, and machine learning computations.
* **Delta Lake:** The optimized storage layer where all processed data resides.
* **Unity Catalog:** The centralized layer managing data governance and security.

**The Lakeflow Framework**
Traditionally, organizations relied on fragmented third-party tools for ingestion, causing operational overhead. Databricks resolves this with **Lakeflow**, a native framework that manages data throughout the Medallion Architecture (moving data from raw *Bronze*, to cleaned *Silver*, to business-ready *Gold* tables).

![[005_assets/Big_Data/Lake flow components.excalidraw|700]]

Lakeflow is divided into three integrated components:


| Component | Role | Function |
| --- | --- | --- |
| **Lakeflow Connect** | Ingestion | Pulls raw data from cloud storage (S3, ADLS), databases, and SaaS apps directly into Delta Lake Bronze tables. |
| **Declarative Pipelines** | Transformation | Uses Delta Live Tables (DLT) to move data from Bronze to Gold, handling automatic dependency resolution and data quality enforcement. |
| **Lakeflow Jobs** | Orchestration | Schedules, monitors, and coordinates the end-to-end execution of pipelines and other Databricks tasks. |


```
---
## Databricks Platform Architecture Overview
Databricks provides a unified data and AI platform built on the following core pillars:
- **Lakeflow** - Data ingestion, transformation, and orchestration
- **Data Processing Engine** - Apache Spark
- **Unified Governance** - Unity Catalog
- **Optimized Storage** - Delta Lake

These components work together to enable scalable, governed, and reliable data pipelines.

### Apache Spark (Data Processing Engine)

Databricks uses Apache Spark as its underlying distributed data processing engine. Spark is responsible for:
- Large-scale data processing
- Batch and streaming workloads
- SQL, ETL, and machine learning computations

All higher-level Databricks features (Lakeflow, Delta Lake, DLT) are executed on top of Spark.

## Lakeflow Overview

Lakeflow is Databricks' unified framework for building and managing data pipelines. It integrates:
- Data ingestion - Lake flow connect
- Data transformation - Lake flow declarative pipeline
- Workflow orchestration - Lake flow Jobs 

All Lakeflow components run on:
- Apache Spark (processing engine)
- Delta Lake (storage layer)
- Unity Catalog (governance layer)

### Lakeflow Components

Lakeflow consists of three core components:
1. **Lakeflow Connect** - Data ingestion
2. **Lakeflow Declarative Pipelines** - Data transformation using Delta Live Tables (DLT)
3. **Lakeflow Jobs** - Workflow orchestration and scheduling

![[005_assets/Big_Data/Lake flow components.excalidraw|700]]


**Background**
Traditionally, organizations relied on 
- multiple third-party ingestion tools
- custom-built ingestion frameworks
- separate tools for different data sources. 

This led to 
- tool sprawl
- increased operational overhead
- inconsistent governance.

**Lakeflow Connect**

Lakeflow Connect provides a native Databricks solution for ingesting data from multiple sources into Delta Lake.
Supported source types include:
* Cloud storage (ADLS Gen2, Amazon S3, Databricks Volumes)
* Databases
* SaaS applications

All ingested data lands in Delta Lake Bronze tables.

Lakeflow pipelines typically follow the Medallion Architecture:
* **Bronze** - Raw ingested data
* **Silver** - Cleaned and enriched data
* **Gold** - Aggregated, business-ready data
* **Consumers** - BI, analytics, ML, applications

### Lakeflow Declarative Pipelines (Delta Live Tables)

Delta Live Tables (DLT) are used for transforming data between Bronze -> Silver -> Gold.
Key characteristics:
* Declarative transformation definitions
* Built-in data quality enforcement
* Automatic dependency resolution
* Native Delta Lake integration

### Lakeflow Jobs
Lakeflow Jobs are used to:
* Schedule ingestion and transformation pipelines
* Orchestrate end-to-end workflows
* Monitor execution and failures

Jobs integrate ingestion, DLT pipelines, and other Databricks tasks into a single workflow.
