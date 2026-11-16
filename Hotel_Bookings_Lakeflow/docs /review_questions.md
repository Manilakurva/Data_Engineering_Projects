# Review Questions and Answers – Hotel Booking Project

## 1. Why do we preserve raw data in the Bronze layer?

We preserve raw data in the Bronze layer to maintain the original source records without changing them. It helps with data traceability, auditing, debugging, and reprocessing whenever required.

## 2. Why is a cancelled booking not considered invalid data?

A cancelled booking is a valid business event because the customer originally made a booking and later cancelled it. It is not a data-quality issue. Therefore, cancelled bookings are retained in the Silver layer for cancellation analysis.

## 3. Why do we use booking_id to identify duplicate bookings?

We use `booking_id` because it uniquely identifies an individual booking. If the same booking ID appears multiple times, it can indicate duplicate booking records that need to be investigated and handled.

## 4. Why can't guest_id be used as a unique booking key?

A guest can make multiple bookings at different hotels or on different dates. Therefore, the same guest ID can appear in multiple records. It cannot uniquely identify an individual booking.

## 5. How does DATEDIFF calculate the number of nights?

`DATEDIFF()` calculates the difference between two dates.

In this project, it calculates the number of nights between the check-in and check-out dates.

**Formula:**

```sql
DATEDIFF(check_out_date, check_in_date)
```

**Example:**

- Check-in date: 2025-04-20
- Check-out date: 2025-04-23
- Number of nights: 3

## 6. Why do we use COALESCE() for missing values?

`COALESCE()` returns the first non-NULL value from the given expressions.

In this project, we use it to handle missing city and booking channel values by replacing them with `Unknown`.

We also use it to calculate missing nights and total amounts when the required source values are available.

**Example:**

```sql
COALESCE(city, 'Unknown')
```

## 7. Why should blank guest ratings remain NULL?

A missing guest rating does not mean the customer gave a zero rating. Keeping it as NULL represents missing information accurately and avoids introducing incorrect values into the dataset.

## 8. What is the difference between total amount and net revenue?

**Total Amount:** The total booking amount calculated using the number of nights and room rate.

**Net Revenue:** The amount remaining after deducting the discount from the total amount.

**Formulas:**

```text
Total Amount = Nights × Room Rate

Net Revenue = Total Amount - Discount Amount
```

Only completed bookings are included in the Gold revenue calculations.

## 9. What is Average Daily Rate (ADR)?

ADR stands for Average Daily Rate. It represents the average revenue earned per occupied room night.

**Formula:**

```text
ADR = Net Revenue / Room Nights
```

It helps hotel management understand the average revenue generated from completed room stays.

## 10. Why are cancelled bookings excluded from revenue?

Cancelled bookings do not represent completed hotel stays. Therefore, they should not contribute to actual earned revenue or completed room-night calculations.

However, cancelled bookings are retained in Silver for booking and cancellation analysis.

## 11. What is the purpose of the rejected table?

The rejected table stores booking records that fail important data-quality or business validation rules.

In this project, records with invalid stay dates or negative room rates are moved to `silver_bookings_rejected`.

It also helps identify rejection reasons and investigate invalid records without losing their details.

## 12. How do we verify data reconciliation?

We verify data reconciliation by comparing the Bronze, clean Silver, rejected, and duplicate record counts.

**Formula:**

```text
Bronze Rows = Clean Rows + Rejected Rows + Duplicate Rows Removed
```

**Project reference results:**

| Metric | Record Count |
|---|---:|
| Bronze Records | 1,316 |
| Clean Records | 1,269 |
| Rejected Records | 21 |
| Duplicate Records Removed | 26 |
| Total Accounted Records | 1,316 |

**Calculation:**

```text
1,269 + 21 + 26 = 1,316
```

The reconciliation status is PASSED when the actual counts satisfy this formula and the record categories do not overlap.

## 13. What is the purpose of a materialized view?

A materialized view stores the results of a query. It helps provide prepared and aggregated data for reporting and analysis.

In this project, materialized views are used in the Silver and Gold layers to create cleaned booking records and business summary tables.

## 14. What is the difference between a streaming table and a materialized view?

| Streaming Table | Materialized View |
|---|---|
| Designed for incremental data processing | Stores the results of a query |
| Processes incoming records | Refreshes results when underlying data changes |
| Commonly used for ingestion | Commonly used for transformations and aggregations |
| Used for the Bronze layer in this project | Used for Silver and Gold outputs |

In this project, `bronze_hotel_bookings` is a streaming table, while the Silver and Gold outputs are materialized views.

## 15. Which booking channel has the highest cancellation rate?

The booking channel with the highest cancellation rate is identified using the `gold_channel_summary` table.

**Formula:**

```text
Cancellation Rate (%) =
(Cancelled Bookings / Total Bookings) × 100
```

The SQL query below identifies the channel with the highest cancellation rate:

```sql
SELECT
    booking_channel,
    total_bookings,
    cancelled_bookings,
    ROUND(
        cancelled_bookings * 100.0 / total_bookings,
        2
    ) AS cancellation_rate
FROM hospitality_catalog.bookings.gold_channel_summary
WHERE total_bookings > 0
ORDER BY cancellation_rate DESC
LIMIT 1;
```

The actual booking channel name should be added after running the query in Databricks.

---

## Conclusion

These 15 review questions cover the important concepts used in the Hotel Booking Data Engineering project, including:

- Medallion Architecture
- Bronze, Silver, and Gold layers
- Data cleaning and validation
- SQL functions
- Hotel booking business rules
- Revenue and ADR calculations
- Data reconciliation
- Streaming tables and materialized views
- Business analysis
