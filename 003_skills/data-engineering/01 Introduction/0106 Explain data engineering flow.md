---
up:
  - "[[003_skills/data-engineering/01 Introduction/01 Introduction|01 Introduction]]"
down:
prev:
topic: false
question: Explain data engineering flow?
---
# Explain data engineering flow

> [!Summary] Summary
> 
> 1. **Source**- Source of data which can be structured, semi- structured, unstructured, streaming.
> 2. **Ingestion** - It pulls the data from various source and ingest the data into data lake-scoop, Azure data factory, Aws glue
> 3. **Datalake** - It stores raw data. Hadoop, Aws S3, Azure data lake storage (ADCS) are used as storage
> 4. **Processing** -The processing - cleaning, transformation on data are performed on this data. Mapreduce, spark, data bricks, synapse, Athenae, redshift are used as processing layer
> 5. **Serving layer**- data warehouse is used as serving layer, Hive, HBase, Azure SQL/Synapse/ AWS RDS are used as serving layer
> 6. **Visualization** - Power bi, tableau are used as visualization layer
> 


![[001_Meta/media/data-engineering-flow.svg]]

## 1. Data Sources
These are the origins of your data. In a real-world scenario, these aren't uniform.
- **Structured:** SQL databases (PostgreSQL, Oracle).
- **Semi-structured:** JSON files, NoSQL databases (MongoDB).
- **Unstructured:** PDF documents, images, or audio files.
- **Streaming:** Real-time clickstream data or IoT sensor readings.

## 2. Ingestion Framework
This is the "vacuum" of the pipeline. It uses **Connectors** to pull data from the sources.
- **Extract:** It authenticates with the source and grabs the data.
- **Tools:** Airbyte, Fivetran, or custom Python scripts using APIs.

## 3. Data Lake
Think of this as your **Raw Storage**. Data is dropped here in its original state before any major changes are made.
- **Why?** It acts as a safety net. If your processing logic changes later, you can always go back to the "source of truth" in the lake.
- **Technologies:** Amazon S3, Azure Data Lake Storage (ADLS), or Google Cloud Storage.

## 4. Processing
This is the "factory" where the raw data is refined.
- **Cleaning:** Removing duplicates or fixing broken records.
- **Transformation:** Converting currencies, masking sensitive information (PII), or joining different datasets together.
- **Techniques:** You might use **Batch processing** (nightly updates) or **Stream processing** (live updates).
- **Tools:** Apache Spark, Databricks, or dbt (Data Build Tool).

## 5. Serving Layer
Once the data is clean and aggregated, it moves to the serving layer, usually a **Data Warehouse**.
- **Optimization:** The data is organized into tables (schemas) that are specifically indexed to make reading very fast for the end-user.
- **Technologies:** Snowflake, Google BigQuery, or Amazon Redshift.

## 6. Visualization & Reports
This is the final destination where business value is created.
- **BI Tools:** Analysts use tools like **Tableau, Power BI, or Looker** to create dashboards.
- **End Goal:** This layer answers questions like _"What were our total sales in Q3?"_ or _"Which region has the highest customer churn?"_
