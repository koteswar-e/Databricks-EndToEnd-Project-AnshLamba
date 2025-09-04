# Databricks-EndToEnd-Project

***

# Project Documentation: End-to-End Azure Databricks Data Platform

## Project Overview
This project demonstrates the construction of a **real-time, scalable data platform** on Azure Databricks, adhering to best practices for data engineering. The solution covers incremental data loading, data quality enforcement, advanced transformations using PySpark and Python OOP, dimensional modeling (Star Schema) with Slowly Changing Dimensions (SCD Type 1 and Type 2), and orchestration using Databricks Workflows. The primary goal is to master Azure Databricks for real-world scenarios and interview preparation.

***

## Architectural Design: Medallion Architecture

The project employs a **Medallion Architecture** to logically separate data based on quality and transformation level:

- **Bronze Layer (Raw Data):**
  - Ingests raw data as-is from source systems with minimal or no transformations.
  - Focuses on persistent storage of original data for auditing and reprocessing.
- **Silver Layer (Enriched/Cleaned Data):**
  - Transforms and cleans data from the Bronze layer.
  - Applies business rules, standardizes formats, and enriches data.
  - Serves as a reliable source for downstream analytical applications.
- **Gold Layer (Curated/Modeled Data):**
  - Structures refined data into a dimensional model (Star Schema: fact and dimension tables).
  - Optimized for analytical queries and reporting.

***

## Infrastructure Setup and Prerequisites

- **Azure Account:** Free Azure account with $200 in credits.
- **Azure Data Lake Storage (ADLS Gen2):**
  - Hierarchical namespace enabled on a Blob Storage account.
  - **Containers:**
    - `source`: Raw incoming Parquet files.
    - `bronze`: Raw data after initial ingestion.
    - `silver`: Cleaned and transformed data.
    - `gold`: Curated data in Star Schema.
    - `metastore`: Unity Catalog’s managed table storage.
- **Azure Databricks Workspace:** Main processing environment for Spark workloads.
  - **Access Connector for Azure Databricks:** Enables Databricks workspace access to ADLS.
- **Unity Catalog:**
  - **Metastore:** Manages metadata for governance.
  - **Credentials:** Uses access connector ID to manage ADLS access.
  - **External Locations:** Logical mapping for each Medallion layer in ADLS.
  - **Catalogs and Schemas:** Organizes bronze, silver, gold layers.

***

## Bronze Layer: Data Ingestion

- **Source Data:** Sourced from GitHub, stored as Parquet in the `source` ADLS container.
- **Incremental Loading (Autoloader):**
  - Uses Spark Structured Streaming for continuous ingestion.
  - **Autoloader:** Processes new data files automatically.
  - **Idempotency:** Managed by `checkpoint_location` for exactly-once processing.
  - **Schema Evolution:** Automates schema changes, using `_rescued_data` for malformed columns.
  - **Dynamic Notebooks:** Parameterized for loading multiple tables.
- **Static Data Ingestion:** No-code ingestion for reference tables (e.g., regions).
- **Output Format:** Data in Bronze stored in Parquet format.

***

## Silver Layer: Data Transformation and Enrichment

- **Technology:** Extensive use of PySpark functions and Python OOP.
- **Transformations:**
  - Drop `_rescued_data` column if not needed.
  - Date/Time conversions using functions like `to_timestamp`, `year`.
  - String manipulations with `split`, `concat`.
  - Aggregations with `group by`, `count`.
  - Window functions: `dense_rank`, `rank`, `row_number`.
- **Code Reusability:** Python classes encapsulate transformation logic.
- **Unity Catalog Functions:** UDFs registered for global or session reuse.
- **Output Format:** Silver data stored in Delta format (ACID, schema enforcement, time travel).

***

## Gold Layer: Data Modeling (Star Schema)

### SCD Type 1 (Customers Dimension)
- **Concept:** Overwrites existing records; keeps only the latest.
- **Implementation:** PySpark operations handle initial and incremental loads.
- **Key Steps:**
  - Duplicate removal (using natural key).
  - Surrogate key generation (`monotonically_increasing_id()`).
  - Upsert Logic via Delta `MERGE`.
  - Metadata columns: `create_date`, `update_date`.
  - Output: Delta table.

### SCD Type 2 (Products Dimension)
- **Concept:** Maintains history of changes with `start_at` and `end_at` columns.
- **Implementation:** Utilizes Delta Live Tables (DLT).
- **DLT Features:**
  - Declarative ETL using decorators (`@dlt.table`, `@dlt.view`).
  - `dlt.apply_changes` simplifies SCD Type 2.
  - Data quality enforced via DLT expectations.
- **Cluster Management:** Ensures proper resource allocation for DLT jobs.

### Fact Table (Orders)
- **Purpose:** Stores transactions linked to dimension tables via surrogate keys.
- **Implementation:**
  - Reads refined data from Silver layer.
  - Integrates dimension keys with joins.
  - Keeps relevant measures and drops natural keys.
  - Upsert logic via Delta `MERGE`.
- **Output:** Delta table.

***

## Orchestration: Databricks Workflows (Jobs)

- **Parent Pipeline:** Master workflow to handle all ETL stages.
- **Task Dependencies:**
  - Parameters notebook runs first (sets dynamic parameters).
  - Bronze Autoloader for incremental loading.
  - Silver notebooks (orders, customers, products) run in parallel post-Bronze.
  - Gold dimension notebooks (SCD Type 1, SCD Type 2) execute after Silver.
  - Fact table runs last, after both Gold dimension tables.
- **Dynamic Execution:** Uses parameterized notebooks and process loops for scalability.

***

## Data Warehousing and BI Integration

- **Databricks SQL Warehouse:** Optimized for serverless SQL workloads on the Gold layer.
- **SQL Editor:** Interactive SQL queries and basic visualizations.
- **Partner Connect:** One-click integration with Power BI, Tableau, etc., using downloadable configuration files for instant connectivity and dashboarding.
