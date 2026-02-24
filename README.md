# Anime Analytics Data Warehouse

An end-to-end **Medallion Architecture** data warehouse and ETL pipeline designed to ingest, transform, and model large-scale media datasets using **Databricks**, **Apache Spark**, and **Delta Lake**.

## Overview
This project demonstrates a production-ready data engineering lifecycle. Moving beyond basic ETL, I designed a multi-stage pipeline that processes CSV data into a highly optimized **Star Schema**. The architecture emphasizes **Idempotency**, **Fault Tolerance**, and **Schema Evolution**, ensuring the pipeline is resilient to failures and handles complex many-to-many relationships (Staff, Genres, and Studios).

## Stack
* **Platform:** Databricks (Unity Catalog & Volumes)
* **Storage Format:** Delta Lake (ACID Transactions, Time Travel)
* **Ingestion:** Databricks Auto Loader (`cloudFiles`)
* **Processing Engine:** Apache Spark (PySpark & SparkSQL)
* **Architecture:** Medallion (Bronze, Silver, Gold)

## Data Architecture & Pipeline

### 1. Bronze Layer (Raw Ingestion)
* **Pattern:** Utilized **Auto Loader** for incremental ingestion from cloud storage volumes.
* **Engineering Solution:** Implemented custom CSV parsing logic with an `escape` option to resolve "Column Shifting" caused by nested double-quotes in anime titles—a critical fix that preserved data integrity for 10,000+ shows.
* **Automation:** Developed a Python-based folder organization script to dynamically isolate datasets, preventing cross-table schema contamination.

### 2. Silver Layer (Cleansing & Normalization)
* **Schema Enforcement:** Cast untyped strings into rigorous technical schemas (Decimal, Integer, and Date types).
* **Regex Engineering:** Cleaned "stuttering" data artifacts (e.g., "Action Action") and removed administrative prefixes using **Regular Expressions**.
* **Granular Normalization:** Used `split()` and `explode()` functions to normalize many-to-many relationships, transforming comma-separated roles (e.g., "Director, Storyboard") into individual, searchable records.

### 3. Gold Layer (Analytics & Star Schema)
* **Modeling:** Designed a **Star Schema** optimized for high-performance analytical querying and BI tools.
* **Fact Table:** `fact_anime_performance` (Metrics: Score, Rank, Popularity, Members).
* **Dimension Tables:** `dim_anime_gold` (Denormalized with genre lists) and `studio_performance_denormalized`.
* **Business Logic:** Pre-aggregated complex joins to allow sub-second querying of studio performance and genre trends.

### 4. Environment & Volume Setup
* **Catalog:** Create a Unity Catalog named `anime_warehouse`.
* **Schemas:** Create three schemas: `bronze`, `silver`, and `gold`.
* **Volume Storage:** Within the `default` schema, create a Volume named `raw_files`.
* **Manual Directory Creation:** Inside the `raw_files` volume, manually create two top-level folders:
    * `/raw_data/` (Upload all raw Kaggle CSVs here).
    * `/checkpoints/` (Leave empty; this stores Auto Loader metadata).

### 5. Pipeline Execution
1.  **Run Folder Organization Script:** Execute the Python script to move CSVs from the root `/raw_data/` into dataset-specific sub-folders (e.g., `/raw_data/anime/`).
2.  **Run Ingestion Notebook:** Execute the Bronze ingestion cell. This uses Auto Loader to read from the Volume sub-folders and write to `anime_warehouse.bronze`.
3.  **Run Transformations:** Execute the Silver and Gold notebooks sequentially to perform data cleaning and dimensional modeling.
