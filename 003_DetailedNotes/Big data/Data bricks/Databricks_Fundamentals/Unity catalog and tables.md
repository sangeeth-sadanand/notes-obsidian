---
up:
  - "[[003_DetailedNotes/Big data/Data bricks/Databricks_Fundamentals/Databricks_Fundamentals|Databricks_Fundamentals]]"
tags:
  - DataBricks/fundamentals
index: 3
type: topic
---
# Unity catalog and tables

```ad-summary
collapse: true 
title: Summary

## Metastore Evolution

### Legacy Hive
- Isolated to individual workspaces and uses a 2-level namespace (`database.table`).

### Unity Catalog (Modern)
- A regional framework that uses a 3-level namespace (`catalog.schema.table`) to improve organization and governance across teams
- It auto-provisions a default catalog (matching the workspace name) containing a `default` schema for user tables and an `information_schema` for system metadata. 
- All workspaces in the same region share the same metastore for consistent security and performance.

## Table Fundamentals

Every table is split into two components:
1. **Data:** The actual files (e.g., Delta, Parquet) residing in cloud storage.
2. **Metadata:** The schema, datatypes, and file pointers cataloged in the **metastore**.
### Table Types

### Managed Tables (Default)
- Databricks fully manages both the data storage and the metadata. 
- They are highly optimized (defaulting to Delta format). 
- **Drop behavior:** Deleting the table permanently removes both the data and the metadata.
### External Tables
- You control the cloud storage location, while Databricks only manages the metadata. 
- This requires granting explicit access credentials to the storage account. 
- **Drop behavior:** Deleting the table removes only the metadata; your data files remain intact.
### Foreign Tables
- References data sitting entirely in an external system (like a relational database). 
- Data is retrieved at runtime. 
- **Drop behavior:** Deleting removes only the metadata.
  
## Unity Catalog Storage Structure
Databricks Unity Catalog provides a unified governance solution for data and AI.

### Hierarchy

![[005_assets/Big_Data/UnityCatalogStructure.excalidraw|UnityCatalogStructure.excalidraw]]

### Data Storage Types
1. **Managed Tables:** Databricks completely manages the physical storage location. Primarily used to store structured data.
2. **External Tables:** The user specifies an external path (e.g., ADLS, S3, GCP). Highly useful when you require full administrative control over the raw storage location.
3. **Volumes:** Logical governance objects used to store and manage non-tabular data (unstructured, semi-structured, or raw structured files).

```
## Metastore Evolution

### The Legacy Hive Metastore

* **Architecture:** Each workspace had its own isolated Hive metastore.
* **Namespace:** Hive uses a 2-level name spacing: `<database>.<table_name>` or `<schema>.<table_name>`.

### Unity Catalog (Modern Framework)

* **Provisioning:** In modern workspaces, a Unity Catalog is provisioned automatically, providing an improved metadata and governance framework.
* **Namespace:** Unity Catalog introduces a 3-level name spacing: `catalog.schema.table-name`. This structure enhances organization, discoverability, and governance across multiple teams.
* **Regional Behavior:**
	* Unity Catalog is regional. When you create the first workspace in a region, it creates a single metastore for that region.
	* Any subsequent workspaces created in the same region will be automatically attached to the same metastore. If you create a workspace in a different region, Databricks will create a new metastore for that region.
	* *Why Regional?* Since the metastore includes security configurations, permissions, and governance settings, keeping the same region improves both performance and compliance.

* **Default Catalog:**
	* Whenever a new Unity Catalog enabled workspace is created, a default catalog with the same namespace as the workspace is created.
	* **Default schema:** Created inside each catalog, where user-defined tables reside.
	* **Information_schema:** Created inside each catalog, containing system-level metadata about tables, columns, and objects.

## Table Fundamentals

Every table in Databricks consists of two parts:

* **Data:** The actual data stored in files, often Parquet or Delta. Data resides in a cloud storage account.
* **Metadata:** Contains information about the table, such as schema definition, column name, datatype, file pointer, and table property.
* **Metastore:** Acts as the cataloging and organizing system for tables where the metadata is stored.


### Table Types

There are three types of tables in Databricks: foreign, managed, and external tables.

#### Foreign Tables

* **Function:** References data stored in an external system such as a relational database.
* **Storage:** Metadata resides in the metastore, while the data continues to remain in the external system.
* **Access:** Querying retrieves data at runtime.
* **Drop Behavior:** Deleting deletes the metastore object only.

#### Managed Tables

* **Function:** This is the default table. Databricks manages both table metadata and data files.
* **Storage:** Data is stored automatically in a Unity Catalog metastore and a stored location that Unity Catalog manages.
* **Use Case:** Ideal when you want Databricks to handle data storage. Perfect for internal datasets used within the workspace, or if you don't want fine-grained control over the file location.
* **Drop Behavior:** Dropping a managed table removes **both** metadata and data.
* **Variant - Managed Table with External Location:** These are still managed tables, but their underlying storage is an external location instead of workspace-managed storage. To create this, you must first create storage credentials, then create the external location.

#### External Tables

* **Function:** For external tables, you choose the storage account, containers, and folders.
* **Storage:** You store metadata in the metastore but keep the data files in a cloud storage location that you control.
* **Access Setup:** Databricks requires explicit permission to access this storage. In Azure, this is done by creating an access connector (managed identity), granting it access to the target storage account (e.g., Storage Blob Data Contributor), and registering it inside Databricks as a storage credential.
* **Drop Behavior:** When you drop the table, it drops **only** the metadata and keeps the files intact.

### Managed vs. External Table Comparison

| Feature | Managed | External |
| --- | --- | --- |
| **Governance** | Unity Catalog manages both metastore and data schema | Unity Catalog controls table-level access |
| **Storage Location** | Can create without a location. Data is stored automatically in catalog/storage location | Requires explicit location in cloud storage |
| **Data Lifecycle** | Drops both metadata and data | Drops only metastore, data remains |
| **Optimizations** | Default format is Delta Table. Fully optimized (auto compaction, better performance, reduced cost) | Supports Delta lake, CSV, JSON, Parquet, ORC, AVRO, Text |


## Unity Catalog Storage Structure
Databricks Unity Catalog provides a unified governance solution for data and AI.

### Hierarchy
![[005_assets/Big_Data/UnityCatalogStructure.excalidraw|UnityCatalogStructure.excalidraw]]

### Data Storage Types
1. **Managed Tables:** Databricks completely manages the physical storage location. Primarily used to store structured data.
2. **External Tables:** The user specifies an external path (e.g., ADLS, S3, GCP). Highly useful when you require full administrative control over the raw storage location.
3. **Volumes:** Logical governance objects used to store and manage non-tabular data (unstructured, semi-structured, or raw structured files).


## Add Storage Location for External and Managed Tables in Azure Databricks

### **Phase 1: Configure Azure Resources (Azure Portal)**

**Step 1: Create an Azure Data Lake Storage Gen2 Account**
1. In the Azure Portal, create a new **Storage account**.
2. **Critical Step:** In the Advanced tab, ensure you check **Enable hierarchical namespace**. This makes it an ADLS Gen2 account, which Databricks requires.
3. Once created, go to **Containers** (under Data storage) and create a new container (e.g., `databricks-data`).

**Step 2: Create an Access Connector (Managed Identity)**
The Access Connector is an Azure resource that acts as an identity for Databricks.
1. In the Azure Portal, search for **Access Connector for Azure Databricks** and click Create.
2. Select your subscription, resource group, and region (ensure it matches your storage account region).
3. Name the connector (e.g., `dbx-access-connector`) and click **Review + Create**.
4. Once deployed, go to the resource's overview page and copy the **Resource ID** (it looks like `/subscriptions/.../providers/Microsoft.Databricks/accessConnectors/...`). You will need this later.

**Step 3: Grant the Connector Access to Your Storage**
1. Navigate back to your **Storage account**.
2. Go to **Access Control (IAM)** on the left menu.
3. Click **+ Add > Add role assignment**.
4. Search for and select the **Storage Blob Data Contributor** role (this allows reading and writing data). Click Next.
5. Under "Assign access to", select **Managed identity**.
6. Click **+ Select members**, choose "Access connector for Azure Databricks", and select the connector you created in Step 2.
7. Click **Review + assign**.

---

### **Phase 2: Configure Databricks (Databricks Workspace)**

**Step 4: Create a Storage Credential in Unity Catalog**
This step tells Databricks about the Managed Identity you just created.
1. Log into your Databricks workspace as an admin.
2. Click **Catalog** in the left sidebar to open the Catalog Explorer.
3. At the bottom of the left pane, click **External Data** > **Credentials**.
4. Click **Create credential** > **Storage credential**.
5. Set the Credential Type to **Azure Managed Identity**.
6. Name the credential (e.g., `my-azure-mi-cred`).
7. Paste the **Access Connector Resource ID** you copied in Step 2.

**Step 5: Create an External Location**
This step maps a specific storage path to the credential, making it usable for your users.
1. Still in the Catalog Explorer, click **External Data** > **External Locations**.
2. Click **Create location**.
3. Name your location (e.g., `adls_data_location`).
4. In the URL field, enter your container path using the `abfss://` format:
   `abfss://<container-name>@<storage-account-name>.dfs.core.windows.net/`
5. Select the **Storage credential** you created in Step 4.
6. Click **Create**.

---

### **Phase 3: Create Tables in Unity Catalog**

#### **Option A: Create an External Table**
An External Table stores data in a location you explicitly define. Dropping the table removes the metadata but **keeps the underlying data files intact** in your ADLS Gen2 container.

**Step 1: Execute the CREATE TABLE statement**
Open a Databricks Notebook or the SQL Editor and define the specific `LOCATION` that falls under your authorized External Location:

```sql
CREATE TABLE my_catalog.default.my_external_table (
  id INT,
  event_name STRING,
  event_timestamp TIMESTAMP
)
USING DELTA
LOCATION 'abfss://<container-name>@<storage-account-name>.dfs.core.windows.net/external_zone/my_external_table/';
```

#### **Option B: Create a Managed Table**

A Managed Table uses the default storage location defined at the Catalog or Schema level. Dropping the table **deletes the underlying data files permanently** from your ADLS Gen2 container.
**Step 1: Assign a Managed Location to a Catalog or Schema** You must assign a dedicated sub-directory of your External Location as the default managed storage path.
1. In Catalog Explorer, click **Catalogs** and click **Create Catalog** (or select an existing catalog and click **Create Schema**).
2. Name the catalog/schema (e.g., `managed_catalog`).
3. Check the box to specify a **Managed location** or **Storage location**.
4. Enter a sub-directory of the `abfss://` URL authorized in Step 5 (e.g., `abfss://<container-name>@<storage-account-name>.dfs.core.windows.net/managed_zone/`).
5. Click **Create**.
##### Create Catalog with External Location
```sql
CREATE CATALOG my_managed_catalog
  MANAGED LOCATION 'abfss://<container-name>@<storage-account-name>.dfs.core.windows.net/managed_catalog_zone/'
  COMMENT 'Catalog with a dedicated managed storage location in ADLS Gen2';
```

##### Create Schema with External Location

```sql
-- First, ensure you are using the correct catalog
USE CATALOG my_catalog;

-- Create the schema with its own managed location
CREATE SCHEMA my_managed_schema
  MANAGED LOCATION 'abfss://<container-name>@<storage-account-name>.dfs.core.windows.net/managed_schema_zone/'
  COMMENT 'Schema with its own dedicated managed storage location';
```

**Step 2: Execute the CREATE TABLE statement** Create the table within the managed namespace. Omit the `LOCATION` clause, and Databricks will automatically place and manage the files.
SQL
```sql
CREATE TABLE managed_catalog.default.my_managed_table (
  employee_id INT,
  department STRING,
  hire_date DATE
) 
USING DELTA;
```