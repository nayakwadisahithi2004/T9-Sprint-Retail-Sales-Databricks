# Retail Sales Data Warehouse – ETL & Data Quality Validation

## About This Project

This project was built to create a complete Retail Sales Data Warehouse pipeline that can handle both initial and incremental data loads in a structured and reliable way. The main focus was not only loading data into a warehouse, but also validating data quality, maintaining historical records, handling invalid records, and automating the complete workflow.

The solution was designed using a layered lakehouse approach, where data moves from source storage into Bronze, Silver, and Gold processing layers before becoming analytics-ready.

At every stage, checks were added to make sure the data is accurate, clean, and consistent.

---

## Project Goal

The goal of this project is to build a practical end-to-end ETL pipeline that can:

* Receive source files from upstream systems
* Store raw copies for traceability
* Archive older files when new incremental files arrive
* Clean and standardize incoming data
* Separate invalid records for quality analysis
* Build dimension and fact tables for reporting
* Track pipeline execution through audit logging
* Run validation checks after every load
* Automate the complete process through workflow orchestration

This project reflects how modern data engineering pipelines are designed in real business environments.

---

## Data Used

The pipeline processes four source datasets:

* Customers
* Products
* Stores
* Sales Transactions

These datasets are used to build:

* **DimCustomer** (with historical tracking using SCD Type 2)
* **DimProduct**
* **DimStore**
* **FactSales**

---

## Storage Design

To keep the pipeline organized, storage was divided into separate zones:

### SFTP Zone

This is the landing area for incoming source files. Every new file first arrives here.

### Raw Zone

A copy of the landed source file is preserved here without any modification. This helps with traceability, reruns, and auditing.

### Archive Zone

Whenever a newer incremental file arrives, the older version is moved to archive instead of being deleted. This keeps a historical record of source files.

### Processed Zone

Cleaned and transformed datasets are stored here in Delta format for downstream use.

This design keeps source, historical, and processed data clearly separated.

---

## Bronze Layer – Raw Ingestion

The Bronze layer is the first processing layer.

At this stage:

* Source CSV files are read from storage
* Tables are created over raw files
* Original source structure is preserved
* No business transformations are applied
* Row counts are validated after loading

The purpose of Bronze is to act as the raw ingestion layer and provide a reliable source for downstream processing.

---

## File Archival Logic

One important requirement of this project was handling incremental files properly.

To solve this:

* File names were identified using timestamps
* The latest file was kept active
* Older versions were moved to archive
* This logic was applied consistently across storage zones
* The process was automated using Python

This ensures that only the newest version is processed while historical files remain available for auditing and recovery.

---

## Silver Layer – Data Cleaning

The Silver layer is where data quality improvements happen.

During this stage:

* Extra spaces are removed
* Text values are standardized
* Emails are converted to lowercase
* Names are converted into proper case
* Dates are converted into consistent formats
* Duplicate records are removed
* Invalid rows are filtered out

This layer transforms raw source data into clean and trusted datasets.

---

## Reject Handling

Instead of silently dropping bad records, invalid rows are captured separately.

Examples include:

* Missing mandatory fields
* Invalid product prices
* Missing store region values
* Sales records with invalid quantity
* Missing keys in transaction data

Each rejected row is stored with:

* A reject reason
* A rejection timestamp

This makes data quality issues easy to trace and analyze.

---

## Gold Layer – Warehouse Model

The Gold layer contains the business warehouse model.

It includes:

### DimCustomer

Built with **Slowly Changing Dimension Type 2**, so customer history is preserved whenever address or city changes.

### DimProduct

Stores cleaned product information.

### DimStore

Stores cleaned store information.

### FactSales

Stores transaction-level sales facts and derives total sales amount using quantity and unit price.

This layer is designed for reporting, dashboards, and analytics.

---

## Audit Logging

To monitor pipeline execution, audit logging was added.

The audit log tracks:

* Which layer executed
* Table processed
* Rows read
* Rows loaded
* Rows rejected
* Execution status
* Load timestamp

This provides clear visibility into every ETL run.

---

## Validation Checks

After processing, multiple validation checks were performed:

* Source-to-target row count checks
* Duplicate checks
* Null value checks
* Referential integrity checks
* Reconciliation between Bronze, Silver, and Reject layers
* Fact table validation

These checks ensure confidence in the final warehouse data.

---

## Workflow Automation

To make the pipeline production-ready, the complete ETL process was automated using a workflow pipeline.

Execution flow:

1. Bronze Ingestion
2. File Archival
3. Silver Transformation
4. Reject Handling
5. Gold Loading
6. Audit Logging
7. Validation Checks

This creates a repeatable and reliable end-to-end execution model.

---

## Final Outcome

This project demonstrates the design of a complete data engineering pipeline with:

* Structured storage zones
* Incremental file handling
* Automated archival
* Bronze / Silver / Gold architecture
* Historical customer tracking
* Reject processing
* Audit logging
* Validation framework
* Automated workflow orchestration

Overall, this project helped build a practical understanding of how enterprise ETL pipelines are designed, validated, and automated in real-world environments.
