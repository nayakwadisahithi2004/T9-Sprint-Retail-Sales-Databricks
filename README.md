# Retail Sales Data Warehouse – ETL & Data Quality Validation

## Project Overview
This project implements an end-to-end ETL pipeline for a Retail Sales Data Warehouse using a modern lakehouse architecture.

The solution ingests retail source data (Customers, Products, Stores, and Sales Transactions), processes it through Bronze, Silver, and Gold layers, validates data quality, archives historical files, and orchestrates execution using an automated workflow pipeline.

The project is designed to simulate an enterprise-grade ETL implementation with incremental loads, archival handling, audit logging, reject processing, and validation checks.


## Business Objective
The objective of this project is to:

- Build a Sales Data Warehouse
- Process full and incremental loads
- Maintain historical customer changes using SCD Type 2
- Validate source-to-target data movement
- Reject invalid records
- Maintain audit logs
- Archive previous source files automatically
- Orchestrate complete ETL workflow automatically

## Architecture

Source System  
↓  
SFTP Landing Zone  
↓  
Raw Zone  
↓  
Bronze Layer  
↓  
Silver Layer  
↓  
Gold Layer  
↓  
Validation / Audit Logging  

Historical Files  
↓  
Archive Zone  

## S3 Bucket Structure
bucket_name/
│
├── sftp/
│   ├── customers/
│   ├── products/
│   ├── stores/
│   └── sales/
│
├── raw/
│   ├── customers/
│   ├── products/
│   ├── stores/
│   └── sales/
│
├── archive/
│   ├── sftp/
│   ├── raw/
│   └── processed/
│
└── processed/
    ├── customers/
    ├── products/
    ├── stores/
    └── sales/

## ETL Layers

### Bronze Layer
Purpose:
- Raw ingestion from S3
- Preserve source structure
- No transformations
- Source row validation

Tables:
- customers_raw
- products_raw
- stores_raw
- sales_raw

---

### Silver Layer
Purpose:
- Data cleansing
- Standardization
- Deduplication
- Invalid data filtering

Transformations:
- TRIM()
- LOWER()
- INITCAP()
- CAST()
- TO_DATE()
- ROW_NUMBER()

Tables:
- silver_customers
- silver_products
- silver_stores
- silver_sales

---

### Reject Layer
Purpose:
Capture invalid records separately.

Reject Rules:
- Missing mandatory columns
- Invalid UnitPrice
- Missing Region
- Quantity <= 0
- Missing IDs

Tables:
- rejected_customers
- rejected_products
- rejected_stores
- rejected_sales

---

### Gold Layer
Purpose:
Business-ready warehouse model.

Dimensions:
- DimCustomer (SCD Type 2)
- DimProduct
- DimStore

Fact:
- FactSales

Measures:
- Quantity
- Amount = Quantity × UnitPrice

---

## SCD Type 2 Implementation

DimCustomer maintains historical changes.

When:
- City changes
- Address changes

Process:
- Expire old record
- Set EndDate
- Insert new active record
- Mark IsActive = 1

Benefits:
- Historical tracking
- Full customer change history

---

## Incremental Load Handling

File naming convention:

```text
<filename>_DDMMYYYYHHMMSS.csv
```

Example:

```text
customers_src_06052026120000.csv
```

Logic:
- Latest file remains active
- Previous file moves to archive
- Applied consistently across zones

---

## Archival Automation

Implemented using Python.

Features:
- Scan source folders
- Extract timestamp from filename
- Identify latest file
- Move older files to archive
- Log archival actions
- Handle failures gracefully

---

## Audit Logging

Audit table captures:

- Layer
- TableName
- RowsRead
- RowsLoaded
- RowsRejected
- Status
- LoadTimestamp

Purpose:
- Monitoring
- Operational visibility
- ETL run tracking

---

## Validation Framework

Checks implemented:

### Source to Target Validation
- Row count validation
- Column mapping validation
- Data type validation

### Data Quality Validation
- Duplicate checks
- Null checks
- Invalid record rejection
- Referential integrity checks

### Reconciliation
Rule:

```text
Bronze = Silver + Reject
```

### Fact Validation
- No duplicate TransactionID
- No null surrogate keys
- Valid dimension lookups

---

## Workflow Orchestration

Automated pipeline built using Databricks Workflow.

Execution Order:

1. Bronze Ingestion  
2. Archive Files  
3. Silver Transform  
4. Reject Handling  
5. Gold Load  
6. Audit Logging  
7. Validation Checks  
