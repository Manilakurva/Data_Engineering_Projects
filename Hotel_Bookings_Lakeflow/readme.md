# Hotel Booking Analytics Platform: Medallion Architecture with Lakeflow Spark Declarative Pipelines

An end-to-end hotel booking data engineering pipeline built on Azure Databricks using Lakeflow Spark Declarative Pipelines (SQL) and the Medallion Architecture (Bronze → Silver → Gold).

This project transforms raw hotel booking data into clean, validated, and business-ready datasets. It includes data ingestion, data cleaning, validation, duplicate handling, data quality tracking, reconciliation, and business reporting through Gold tables and dashboards.

# Table of Contents

- Business Problem
- Architecture
- Pipeline Lineage
- Tech Stack
- Dataset
- Repository Structure
- Layer Details
- Data Quality Rules
- Reconciliation
- Business Metrics
- Dashboard
- How to Run
- Business Requirements Coverage
- Key Learnings
- Future Improvements

# Business Problem

Hotels receive booking information through different channels, including websites, mobile applications, travel agents, and walk-in reservations.

However, raw booking data may contain duplicate records, missing values, inconsistent text formats, invalid dates, and incorrect room rates. These issues can affect booking reports, revenue calculations, and business decisions.

The goal of this project is to build a repeatable data engineering pipeline that preserves the original data, cleans and validates booking records, and generates trusted business-level summary tables.

The final output helps understand hotel performance, booking trends, room-type performance, revenue, and cancellation patterns.

# Architecture

The project follows the Medallion Architecture:

`hotel_bookings_raw.csv` (Unity Catalog Volume)  
↓  
**Bronze Layer** – Raw data ingestion and metadata  
↓  
**Silver Layer** – Data cleaning and validation  
↓  
**Gold Layer** – Business-level aggregations  
↓  
**Databricks SQL / AI/BI Dashboard / Power BI**

# Pipeline Lineage

| Layer | Purpose | Key Actions |
|---|---|---|
| Bronze | Preserve raw data | Load CSV, retain source values, add ingestion metadata |
| Silver | Improve data quality | Remove duplicates, standardize text, handle missing values, validate records |
| Gold | Support business reporting | Monthly booking trends, hotel performance, room-type analysis, channel analysis, data quality summary |

## Pipeline Objects

| Object | Type | Description |
|---|---|---|
| `bronze_hotel_bookings` | Streaming Table | Stores raw hotel booking records with ingestion metadata |
| `silver_bookings_clean` | Silver Pipeline Table | Stores cleaned and validated booking records |
| `silver_bookings_rejected` | Silver Pipeline Table | Stores records that fail validation rules |
| `gold_monthly_bookings` | Gold Summary Table | Monthly booking and revenue analysis |
| `gold_hotel_summary` | Gold Summary Table | Hotel-wise booking and revenue performance |
| `gold_room_type_summary` | Gold Summary Table | Room-type performance analysis |
| `gold_channel_summary` | Gold Summary Table | Booking channel and cancellation analysis |
| `gold_data_quality` | Gold Summary Table | Data quality and reconciliation summary |

# Tech Stack

- Azure Databricks
- Lakeflow Spark Declarative Pipelines
- Databricks SQL
- Unity Catalog
- Unity Catalog Volumes
- Delta Lake
- Databricks AI/BI or Power BI

# Dataset

The project uses a synthetic hotel booking dataset named `hotel_bookings_raw.csv`.

**Dataset Information:**

- Total Records: 1,316
- Total Columns: 20
- Number of Hotels: 10
- Number of Cities: 7
- Data Period: 2025
- Room Types: Standard, Deluxe, Suite, Family Room
- Booking Channels: Website, Mobile App, Travel Agent, Walk In
- Booking Status: Completed and Cancelled

**Important Dataset Columns:**

`booking_id`, `guest_id`, `booking_date`, `check_in_date`, `check_out_date`, `nights`, `hotel_name`, `city`, `region`, `room_type`, `booking_channel`, `guests`, `room_rate`, `total_amount`, `discount_amount`, `meal_plan`, `payment_method`, `payment_status`, `booking_status`, `guest_rating`.

The grain of the dataset is one booking per row. The `booking_id` is used to identify unique booking records. A guest can make multiple bookings, so `guest_id` is not a unique booking key.

# Repository Structure

```text
Hotel-Booking-Data-Engineering/
│
├── README.md
│
├── data/
│   └── hotel_bookings_raw.csv
│
├── pipeline/
│   ├── bronze/
│   │   └── bronze_transformation.sql
│   │
│   ├── silver/
│   │   └── silver_transformation.sql
│   │
│   └── gold/
│       └── gold_transformation.sql
│
├── validation/
│   └── reconciliation_queries.sql
│
├── docs/
│   ├── data_quality_rules.md
│   ├── reconciliation.md
│   └── review_questions.md
│
└── images/
    ├── architecture.png
    ├── pipeline_lineage.png
    └── dashboard.png
```

# Layer Details

## Bronze Layer – Raw Data Ingestion

The Bronze layer stores the original hotel booking data.

**Table:** `bronze_hotel_bookings`

**Source:**

```text
/Volumes/hospitality_catalog/bookings/raw_files/
```

**Key Activities:**

- Read the CSV file from the Unity Catalog Volume.
- Load the source records into the Bronze layer.
- Preserve the original source values.
- Add ingestion metadata, such as source filename and ingestion timestamp.
- Maintain the raw data for traceability and future reprocessing.

## Silver Layer – Data Cleaning and Validation

The Silver layer improves data quality by cleaning and validating the Bronze records.

**Tables:**

- `silver_bookings_clean`
- `silver_bookings_rejected`

**Key Activities:**

- Identify duplicate records using `booking_id`.
- Remove unnecessary leading and trailing spaces.
- Standardize text values and capitalization.
- Handle missing city and booking channel values using `Unknown`.
- Calculate missing nights using check-in and check-out dates.
- Calculate missing total amounts using nights and room rates.
- Keep missing guest ratings as NULL.
- Identify invalid dates and negative room rates.
- Separate records that fail validation into the rejected table.
- Retain cancelled bookings as valid records.

## Gold Layer – Business Transformations

The Gold layer contains business-ready summary tables created from the cleaned Silver data.

**Gold Tables:**

| Table | Description |
|---|---|
| `gold_monthly_bookings` | Monthly booking and revenue trends |
| `gold_hotel_summary` | Hotel-wise booking and revenue performance |
| `gold_room_type_summary` | Room-type booking and revenue analysis |
| `gold_channel_summary` | Booking channel performance and cancellation rates |
| `gold_data_quality` | Data quality and reconciliation metrics |

Cancelled bookings are included in booking and cancellation analysis but excluded from net revenue and room-night calculations.

# Data Quality Rules

| Rule | Treatment |
|---|---|
| Completeness | Identify missing values in important fields |
| Standardization | Remove extra spaces and standardize text values |
| Missing City | Replace blank values with `Unknown` |
| Missing Booking Channel | Replace blank values with `Unknown` |
| Missing Nights | Calculate using check-in and check-out dates |
| Missing Total Amount | Calculate using nights and room rate when possible |
| Missing Guest Rating | Retain as NULL |
| Date Validation | Reject records with impossible or invalid dates |
| Business Validation | Reject records with negative room rates |
| Duplicate Handling | Identify duplicate booking records using `booking_id` |
| Booking Status | Retain both Completed and Cancelled bookings |
| Revenue Calculation | Include only completed bookings |

**Important:** Cancelled bookings are not invalid records. They are retained for business analysis but excluded from revenue and room-night calculations.

Details: `docs/data_quality_rules.md`

# Reconciliation

Reconciliation ensures that all records from the Bronze layer are properly accounted for in the Silver layer.

**Formula:**

`Bronze Rows = Clean Rows + Rejected Rows + Duplicate Rows Removed`

**Reference Results:**

| Check | Result |
|---|---:|
| Source CSV Rows | 1,316 |
| Bronze Rows | 1,316 |
| Silver Clean Rows | 1,269 |
| Rejected Rows | 21 |
| Duplicate Rows Removed | 26 |
| Total Accounted Rows | 1,316 |

The reconciliation is valid when rejected records and duplicate records removed are counted separately without overlap.

Details: `docs/reconciliation.md`

# Business Metrics

The following metrics are calculated using the cleaned booking data:

| Metric | Formula |
|---|---|
| Total Bookings | Count of valid booking records |
| Net Revenue | Total Amount − Discount Amount |
| Room Nights | Sum of completed booking nights |
| Average Daily Rate (ADR) | Net Revenue / Room Nights |
| Cancellation Rate | (Cancelled Bookings / Total Bookings) × 100 |
| Monthly Bookings | Booking count grouped by month |

Only completed bookings contribute to revenue and room-night calculations.

# Dashboard

An interactive dashboard is developed using the Gold tables to understand hotel booking performance.

**Dashboard Pages:**

### 1. Overview
- Total Bookings
- Net Revenue
- Room Nights
- Cancelled Bookings
- Monthly Booking Trends

### 2. Hotels, Rooms and Channels
- Hotel-wise Performance
- Room Type Analysis
- Booking Channel Analysis
- Revenue Comparison

### 3. Data Quality
- Bronze Records
- Clean Records
- Rejected Records
- Duplicate Records Removed
- Data Reconciliation

# How to Run

1. Upload `hotel_bookings_raw.csv` to the Unity Catalog Volume.
2. Verify the catalog, schema, volume, and file permissions.
3. Open Azure Databricks.
4. Create a Lakeflow Spark Declarative Pipeline.
5. Add the Bronze, Silver, and Gold SQL transformation files.
6. Configure the target catalog and schema:
   - Catalog: `hospitality_catalog`
   - Schema: `bookings`
7. Configure the pipeline to read the source CSV file from the Unity Catalog Volume.
8. Run the pipeline.
9. Verify the Bronze, Silver, and Gold tables.
10. Execute the reconciliation queries from the `validation/` folder.
11. Query the Gold tables and validate the business metrics.
12. Connect the Gold tables to the dashboard and verify the results.

# Business Requirements Coverage

| ID | Requirement | Implementation |
|---|---|---|
| BR-01 | Raw data ingestion | `bronze_hotel_bookings` |
| BR-02 | Data cleaning and standardization | `silver_bookings_clean` |
| BR-03 | Invalid record handling | `silver_bookings_rejected` |
| BR-04 | Monthly booking analysis | `gold_monthly_bookings` |
| BR-05 | Hotel performance analysis | `gold_hotel_summary` |
| BR-06 | Room-type analysis | `gold_room_type_summary` |
| BR-07 | Booking channel and cancellation analysis | `gold_channel_summary` |
| BR-08 | Data quality visibility | `gold_data_quality` |
| BR-09 | Data reconciliation | `validation/` |
| BR-10 | Business reporting | Interactive Dashboard |

# Key Learnings

Through this project, I learned:

- How to build an end-to-end data engineering pipeline.
- How the Medallion Architecture organizes data into Bronze, Silver, and Gold layers.
- How to ingest raw data using Unity Catalog Volumes.
- How to clean and validate data using Databricks SQL.
- How to identify and handle duplicate and invalid records.
- Why cancelled bookings should be retained for business analysis.
- How to calculate business metrics such as net revenue, room nights, and ADR.
- How to perform data reconciliation across pipeline layers.
- How to create business-ready summary tables for dashboards.

# Future Improvements

- Automate daily hotel booking data ingestion.
- Implement incremental data processing.
- Add more data quality checks and monitoring.
- Create separate guest and hotel dimension tables.
- Add alerts for pipeline failures and data quality issues.
- Improve dashboard reporting with additional business KPIs.

# License

This project is created for learning and internship purposes.
