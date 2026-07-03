# End-to-End Data Engineering Pipeline for Insurance Company

This project implements a **scalable end-to-end data engineering pipeline** for an insurance company to analyse claims data and perform customer segmentation.  
The solution is built using **Azure Data Engineering services** and follows the **Medallion Architecture (Bronze → Silver → Gold)**.

---

## Table of Contents
- [Project Overview](#project-overview)
- [Architecture](#architecture)
- [Smart Policy Data System](#smart-policy-data-system)
- [Services Used](#services-used)
- [Data Ingestion (ADF Pipelines)](#data-ingestion-adf-pipelines)
  - [Full Load vs Incremental Load](#full-load-vs-incremental-load)
  - [Claim Data (Incremental - HWM)](#claim-data-incremental---hwm)
  - [Agent Data (Incremental - HWM)](#agent-data-incremental---hwm)
  - [Branch Data (Full Load)](#branch-data-full-load)
- [Medallion Architecture](#medallion-architecture)
  - [Bronze Layer](#bronze-layer)
  - [Silver Layer](#silver-layer)
  - [Gold Layer](#gold-layer)
- [Reporting](#reporting)
- [Conclusion](#conclusion)

---

## Project Overview

This project builds an **end-to-end data pipeline for an insurance company** to:
- Analyse claims data
- Perform customer segmentation
- Enable business insights for decision-making

The pipeline ensures **data consistency, scalability, and reliability** using Azure services.

---

## Architecture

The system follows a **Medallion Architecture**:
Bronze → Silver → Gold
Raw Data → Cleaned Data → Business-Ready Data

### Architecture Diagram

<img width="666" height="442" alt="architecture" src="https://github.com/user-attachments/assets/f1cb90a4-6f40-46bc-805e-58497588bec2" />

---

## Smart Policy Data System

| Component | Description |
|----------|-------------|
| Source Systems | CSV, JSON, SQL Server |
| Architecture | Bronze → Silver → Gold |
| Data Issues | Inconsistent raw data requiring cleaning |
| Bronze Layer | Raw ingestion layer |
| Silver Layer | Data cleaning & transformation |
| Gold Layer | Curated analytics layer |
| Reporting | Power BI dashboards |

---

## Services Used

| Service | Purpose |
|---------|--------|
| Azure Data Lake Storage (ADLS Gen2) | Data storage (raw + processed) |
| Azure Data Factory (ADF) | Data ingestion & orchestration |
| Azure SQL Database | Source system |
| Azure Databricks | Data processing (Lakehouse) |
| Azure Key Vault | Secret management |
| Azure DevOps (Git) | CI/CD and version control |
| Power BI | Reporting & visualization |

---

## Data Ingestion (ADF Pipelines)

### Full Load vs Incremental Load

| Data Source | Load Type | Method |
|------------|----------|--------|
| Branch | Full Load | ADF Copy Activity |
| Claim | Incremental | High Water Mark (HWM) |
| Agent | Incremental | High Water Mark (HWM) |
| Policy | Incremental | ADF Pipeline |
| Customer | Incremental | After initial full load |

---

## Claim Data (Incremental - HWM)

The Claim pipeline uses a **High Water Mark (HWM)** approach.
### Steps

**1. Get HWM Date**

- Lookup activity reads the last processed timestamp from ADLS.

**2. Get Latest Timestamp**

```sql
SELECT MAX(lastupdatedtimestamp) AS last_update_timestamp
FROM dbo.claim;
```

**3. Extract Incremental Data**

```sql
SELECT *
FROM dbo.claim
WHERE lastupdatedtimestamp >
'@{activity('get hwm from csv').output.firstRow.Prop_0}'
AND lastupdatedtimestamp <=
'@{activity('get max date from table').output.firstRow.last_update_timestamp}';
```

**4. Update HWM**

```sql
SELECT
'@{activity('get max date from table').output.firstRow.last_update_timestamp }'
FROM dbo.claim;
```
- Latest timestamp stored back in ADLS
  
  
**5. Pipeline workflow**


 <img width="2930" height="1058" alt="image" src="https://github.com/user-attachments/assets/fbbf8099-b2bc-4c17-a2f4-87fd1b13e3e0" />

 ## Agent Data (Incremental - HWM)

The Agent pipeline uses a **High Water Mark (HWM)** approach.
### Steps

**1. Get HWM Date**

- Lookup activity reads the last processed timestamp from ADLS.

**2. Get Latest Timestamp**

```sql
SELECT MAX(lastupdatedtimestamp) AS last_update_timestamp
FROM dbo.agent;
```

**3. Extract Incremental Data**

```sql
SELECT *
FROM dbo.agent
WHERE lastupdatedtimestamp >
'@{activity('get hwm from csv').output.firstRow.Prop_0}'
AND lastupdatedtimestamp <=
'@{activity('get max date from table').output.firstRow.last_update_timestamp}';
```

**4. Update HWM**

```sql
SELECT
'@{activity('get max date from table').output.firstRow.last_update_timestamp }'
FROM dbo.agent;
```
- Latest timestamp stored back in ADLS
  
  
**5. Pipeline workflow**

<img width="1463" height="485" alt="Screenshot 2026-07-02 at 3 24 39 pm" src="https://github.com/user-attachments/assets/56d18752-dc35-40e0-ad10-670ac77f7027" />

