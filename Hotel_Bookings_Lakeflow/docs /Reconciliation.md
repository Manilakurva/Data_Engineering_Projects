# Data Reconciliation – Hotel Booking Project

## 1. Overview

Data reconciliation is the process of verifying whether records are correctly processed and accounted for across different layers of a data pipeline.

In this Hotel Booking Analytics project, reconciliation ensures that all source records are properly accounted for after processing through the Bronze and Silver layers.

The source dataset contains **1,316 booking records**. The reconciliation process checks the record counts, duplicate records, rejected records, and revenue calculations.

## 2. Reconciliation Formula

The project follows this reconciliation formula:

```text
Bronze Records = Clean Records + Rejected Records + Duplicate Records Removed
```

### Reconciliation Results

| Metric | Record Count |
|---|---:|
| Source CSV Records | 1,316 |
| Bronze Records | 1,316 |
| Silver Clean Records | 1,269 |
| Rejected Records | 21 |
| Duplicate Records Removed | 26 |
| Total Accounted Records | 1,316 |

### Reconciliation Calculation

```text
1,269 + 21 + 26 = 1,316
```

**Reconciliation Status: PASSED**

All source records are accounted for based on the project reference results.

## 3. Reconciliation Checks

| Check ID | Validation | Expected Result |
|---|---|---|
| REC-01 | Source CSV vs Bronze record count | 1,316 records |
| REC-02 | Silver clean record count | 1,269 records |
| REC-03 | Rejected record count | 21 records |
| REC-04 | Duplicate records removed | 26 records |
| REC-05 | Total record reconciliation | 1,316 records |
| REC-06 | Cancelled booking validation | Retained in Silver |
| REC-07 | Revenue validation | Only completed bookings included |
| REC-08 | Room-night validation | Only completed stays included |

## 4. SQL Validation Queries

The following SQL queries are used to validate record counts across the pipeline.

### 4.1 Bronze Record Count

```sql
SELECT COUNT(*) AS bronze_rows
FROM hospitality_catalog.bookings.bronze_hotel_bookings;
```

Expected result: `1316`

### 4.2 Silver Clean Record Count

```sql
SELECT COUNT(*) AS clean_rows
FROM hospitality_catalog.bookings.silver_bookings_clean;
```

Expected result: `1269`

### 4.3 Rejected Record Count

```sql
SELECT COUNT(*) AS rejected_rows
FROM hospitality_catalog.bookings.silver_bookings_rejected;
```

Expected result: `21`

### 4.4 Duplicate Records Removed

```sql
SELECT
    SUM(record_count - 1) AS duplicate_rows_removed
FROM (
    SELECT
        booking_id,
        COUNT(*) AS record_count
    FROM hospitality_catalog.bookings.bronze_hotel_bookings
    GROUP BY booking_id
    HAVING COUNT(*) > 1
) duplicates;
```

Expected result: `26`

### 4.5 Final Reconciliation

```sql
WITH reconciliation AS (
    SELECT
        (SELECT COUNT(*)
         FROM hospitality_catalog.bookings.bronze_hotel_bookings) AS bronze_rows,

        (SELECT COUNT(*)
         FROM hospitality_catalog.bookings.silver_bookings_clean) AS clean_rows,

        (SELECT COUNT(*)
         FROM hospitality_catalog.bookings.silver_bookings_rejected) AS rejected_rows,

        (SELECT SUM(record_count - 1)
         FROM (
             SELECT booking_id, COUNT(*) AS record_count
             FROM hospitality_catalog.bookings.bronze_hotel_bookings
             GROUP BY booking_id
             HAVING COUNT(*) > 1
         )) AS duplicate_rows_removed
)

SELECT
    bronze_rows,
    clean_rows,
    rejected_rows,
    duplicate_rows_removed,
    clean_rows + rejected_rows + duplicate_rows_removed AS accounted_rows,

    CASE
        WHEN bronze_rows =
             clean_rows + rejected_rows + duplicate_rows_removed
        THEN 'PASSED'
        ELSE 'FAILED'
    END AS reconciliation_status

FROM reconciliation;
```

**Note:** This formula assumes duplicate records are excluded from both the clean and rejected counts. Duplicate counts must represent extra occurrences, not the number of duplicated booking IDs.

## 5. Financial Reconciliation

Financial reconciliation verifies whether the revenue calculated in the Silver layer matches the revenue reported in the Gold layer.

### Silver Net Revenue

```sql
SELECT
    SUM(total_amount - discount_amount) AS silver_net_revenue
FROM hospitality_catalog.bookings.silver_bookings_clean
WHERE booking_status = 'Completed';
```

### Gold Net Revenue

```sql
SELECT
    SUM(net_revenue) AS gold_net_revenue
FROM hospitality_catalog.bookings.gold_monthly_bookings;
```

The Silver and Gold revenue totals should match when both layers follow the same business rules and date filters.

## 6. Business Rules for Reconciliation

- All original records must be preserved in the Bronze layer.
- Duplicate booking records must be identified and handled.
- Invalid booking records must be stored in the rejected table.
- Cancelled bookings must remain in the Silver layer.
- Cancelled bookings must not contribute to net revenue.
- Only completed bookings contribute to room-night calculations.
- Gold summary totals must match the corresponding Silver data.

## 7. Final Outcome

The reconciliation process helps verify the accuracy and completeness of the Hotel Booking Data Engineering pipeline.

It ensures that:

- All source records are accounted for.
- Duplicate records are identified.
- Invalid records are tracked separately.
- Revenue calculations follow business rules.
- Gold summary tables contain consistent business information.

**Final Result:** The reference reconciliation is PASSED with 1,316 total records accounted for.
