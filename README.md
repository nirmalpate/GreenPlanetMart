# GreenPlanetMart
SnowFlake Power BI Project
## 📊 GreenPlanMart Analytics Data Pipeline

This repository contains the necessary configuration, SQL scripts, and a Power BI report file to set up a robust, automated analytical pipeline using Snowflake for data warehousing, Snowpipe for continuous data ingestion, and Power BI for reporting.

### ✨ Key Features

  * **Scalable Data Warehouse:** Centralized data storage and processing using **Snowflake**.
  * **Dimensional Modeling:** Efficient and optimized data structure using **Star Schema** (Facts and Dimensions).
  * **Automated Ingestion:** Near real-time data loading from cloud storage (e.g., S3, Azure Blob, Google Cloud Storage) into Snowflake using **Snowpipe**.
  * **Scheduled Transformation:** Automated data transformation and aggregation using **Snowflake Tasks**.
  * **Business Intelligence:** Interactive and dynamic reporting using **Power BI**.

### 🏗️ Architecture Overview

The pipeline is designed for low-latency data ingestion and transformation, ensuring that the Power BI reports reflect the freshest data available.

1.  **Source:** Raw data files (e.g., CSV, JSON) are dropped into an external Cloud Storage staging area.
2.  **Ingestion (Snowpipe):** Snowpipe continuously monitors the stage and loads new files automatically into a Snowflake **STAGING** layer table.
3.  **Transformation (Snowflake Tasks):** A scheduled Snowflake **TASK** wakes up (e.g., every 5 minutes) to perform ETL/ELT:
      * De-duplication and data quality checks.
      * Transforming the raw data into the optimized **DIMENSION** and **FACT** tables.
4.  **Reporting (Power BI):** Power BI connects directly to the finalized **FACT** and **DIMENSION** tables in Snowflake to generate business insights.

### ⚙️ Prerequisites

To deploy and maintain this project, you will need:

1.  A **Snowflake** Account (with necessary roles and warehouse access).
2.  Access to a **Cloud Storage** service (AWS S3, Azure Blob Storage, or GCP Cloud Storage) for Snowpipe to monitor.
3.  **SnowSQL CLI** or a preferred SQL client for running setup scripts.
4.  **Power BI Desktop** (or Power BI Service) for report development and publishing.

### 🚀 Deployment Guide

#### 1\. Snowflake Setup (Data Model & Schema)

The following high-level steps should be executed in Snowflake:

  * **Database and Schema Creation:**
    ```sql
    CREATE DATABASE GREENPLANMART_DW;
    CREATE SCHEMA STAGING; -- For raw, ingested data
    CREATE SCHEMA ANALYTICS; -- For final dimensional model
    ```
  * **Table Creation:**
      * Create `RAW_SALES` table in the `STAGING` schema.
      * Create **Dimension** tables (e.g., `DIM_DATE`, `DIM_PRODUCT`, `DIM_CUSTOMER`) in the `ANALYTICS` schema.
      * Create the main **Fact** table (e.g., `FACT_SALES`) in the `ANALYTICS` schema.

#### 2\. Automated Ingestion with Snowpipe

  * **Cloud Storage Integration:** Create a Storage Integration object in Snowflake to grant access to your cloud storage.
  * **Stage Creation:** Create an External Stage that points to the data source location (e.g., `s3://greenplanmart-data/sales/`).
  * **Snowpipe Definition:** Create the Snowpipe object to load data from the stage into the raw table.
    ```sql
    CREATE PIPE STAGING.SALES_PIPE AUTO_INGEST=TRUE AS
    COPY INTO STAGING.RAW_SALES
    FROM @<EXTERNAL_STAGE_NAME>
    FILE_FORMAT = (TYPE = CSV ...);
    ```
  * **Notification Setup:** Configure your cloud storage to send file arrival notifications (SQS for AWS, Event Grid for Azure, etc.) to the generated Snowpipe endpoint for immediate loading.

#### 3\. Automated Transformations with Snowflake Tasks

  * **Task Definition:** Create a root task that runs on a schedule and calls a stored procedure to perform the data merge/upsert operations into the dimensional model.
    ```sql
    CREATE TASK ANALYTICS.TRANSFORM_SALES
    WAREHOUSE = <YOUR_WAREHOUSE>
    SCHEDULE = '5 MINUTE'
    WHEN SYSTEM$STREAM_HAS_DATA('STAGING.RAW_SALES_STREAM') -- Optional: only run if new data is available
    AS CALL ANALYTICS.SP_LOAD_FACT_SALES();
    ```
  * **Task Activation:**
    ```sql
    ALTER TASK ANALYTICS.TRANSFORM_SALES RESUME;
    ```

#### 4\. Power BI Reporting

1.  **Connect to Snowflake:** In Power BI Desktop, use the Snowflake connector, pointing to the `ANALYTICS` schema.
2.  **Data Model:** Import the `FACT_SALES` and related `DIM_*` tables. Define the relationships in Power BI (Star Schema).
3.  **Visualization:** Create visualizations to answer business questions. The report file provided, **`GreenPlanMart_Report.pbix`**, contains the completed data model and visualizations.
4.  **Refresh:** Configure the Power BI Gateway for scheduled data refresh from Snowflake (or use DirectQuery mode).
