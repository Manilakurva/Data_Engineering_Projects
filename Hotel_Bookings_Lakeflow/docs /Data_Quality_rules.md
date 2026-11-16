# Data Quality Rules – Hotel Booking Project

## 1. Introduction

Data quality is an important part of the Hotel Booking Analytics Platform.

The raw hotel booking dataset contains 1,316 records with several data-quality issues, such as duplicate bookings, extra spaces, inconsistent capitalization, missing values, impossible stay dates, and negative room rates.

The main objective is to clean and validate the data in the Silver layer while preserving the original records in the Bronze layer.

The project uses SQL transformations and data-quality checks to improve data accuracy and consistency.

## 2. Data Quality Rules

The following rules are applied during the Silver-layer transformation.

| Rule | Problem | Example | Treatment | SQL Concept |
|---|---|---|---|---|
| DQ-01 | Duplicate bookings | Same booking_id appears more than once | Keep only one copy | SELECT DISTINCT |
| DQ-02 | Extra spaces | `"  goa"` | Remove leading and trailing spaces | TRIM() |
| DQ-03 | Mixed capitalization | `"DELUXE"` | Convert text into title case | INITCAP() |
| DQ-04 | Missing city | City is empty | Replace with `Unknown` | COALESCE() |
| DQ-05 | Missing booking channel | Channel is empty | Replace with `Unknown` | COALESCE() |
| DQ-06 | Missing nights | Nights is NULL | Calculate using check-in and check-out dates | DATEDIFF() |
| DQ-07 | Missing total amount | Total amount is NULL | Calculate nights × room rate | COALESCE() |
| DQ-08 | Missing guest rating | Rating is NULL | Keep as NULL | No change |
| DQ-09 | Missing guest ID | Guest ID is NULL | Keep as NULL | No change |
| DQ-10 | Invalid stay dates | Check-out date is before or equal to check-in date | Move record to rejected table | Date validation |
| DQ-11 | Negative room rate | Room rate = -4500 | Move record to rejected table | Room rate validation |

## 3. Data Cleaning Rules – Detailed Explanation

### Rule 1: Duplicate Booking Identification

**Problem:** The same booking ID appears more than once in the dataset.

**Solution:** Identify duplicate booking records and retain only one copy.

**SQL concept:**
`SELECT DISTINCT`

**Expected result:** Duplicate bookings should not appear in the clean Silver table.

### Rule 2: Remove Extra Spaces

**Problem:** Some text columns contain unwanted spaces.

Examples:
- `"  goa"`
- `"DELUXE"`
- `" mobile app "`

**Solution:** Use TRIM() to remove leading and trailing spaces.

**SQL concept:**
`TRIM(column_name)`

### Rule 3: Standardize Text Values

**Problem:** Some values have inconsistent capitalization.

Examples:
- `DELUXE`
- `deluxe`
- `Deluxe`

**Solution:** Use INITCAP() to standardize text into title case.

**SQL concept:**
`INITCAP(TRIM(column_name))`

### Rule 4: Handle Missing City and Booking Channel

**Problem:** Some booking records have blank city or booking channel values.

**Solution:** Replace missing values with `Unknown`.

**SQL concept:**
`COALESCE(column_name, 'Unknown')`

### Rule 5: Calculate Missing Nights

**Problem:** Some records have missing nights values.

**Solution:** Calculate the number of nights using the difference between check-out and check-in dates.

**SQL concept:**
`DATEDIFF(check_out_date, check_in_date)`

Example:

Check-in date: 2025-04-20  
Check-out date: 2025-04-23  
Calculated nights: 3

Use COALESCE() to retain an existing nights value or calculate it when missing.

### Rule 6: Calculate Missing Total Amount

**Problem:** Some bookings have missing total amount values.

**Solution:** Calculate the amount using the number of nights multiplied by the room rate.

**Formula:**

`total_amount = nights × room_rate`

Use COALESCE() to retain an existing total amount or calculate it when missing.

### Rule 7: Handle Missing Guest Rating and Guest ID

**Problem:** Some records have missing guest ratings or guest IDs.

**Solution:** Keep these values as NULL rather than replacing them with artificial values.

A missing guest rating should not be treated as zero because zero is not a valid rating in the dataset.

### Rule 8: Validate Stay Dates

**Problem:** Some bookings have check-out dates before or equal to check-in dates.

**Business Rule:**

`check_out_date > check_in_date`

**Solution:** Move invalid records to `silver_bookings_rejected` and retain the rejection reason.

**SQL concept:**

`EXPECT (check_out_date > check_in_date) ON VIOLATION DROP ROW`

### Rule 9: Validate Room Rate

**Problem:** Some bookings contain negative room rates.

**Business Rule:**

`room_rate >= 0`

**Solution:** Move records with negative room rates to the rejected table.

The rejection reason should identify the negative room rate.

## 4. Special Business Rule: Cancelled Bookings

Cancelled bookings are normal business events and should not be treated as data-quality errors.

**Treatment:**

- Keep completed bookings in Silver.
- Keep cancelled bookings in Silver.
- Count cancelled bookings in booking reports.
- Exclude cancelled bookings from net revenue.
- Exclude cancelled bookings from completed room-night calculations.

Only completed bookings contribute to revenue and room nights in the Gold layer.

## 5. Rejected Records Management

Invalid bookings are stored separately in:

`silver_bookings_rejected`

Each rejected record should contain a reason explaining why it was rejected.

Examples:

| Booking Issue | Rejection Reason |
|---|---|
| Check-out before check-in | Check-out is not after check-in |
| Check-out equal to check-in | Check-out is not after check-in |
| Negative room rate | Room rate is negative |

This helps maintain transparency and makes it easier to investigate data-quality problems.

## 6. Data Reconciliation

After cleaning and validation, reconcile the records across the pipeline.

**Reconciliation Formula:**

`Bronze Rows = Clean Rows + Rejected Rows + Duplicate Rows`

### Reference Results

| Metric | Expected Count |
|---|---:|
| Bronze Rows | 1,316 |
| Clean Bookings | 1,269 |
| Rejected Bookings | 21 |
| Duplicates Removed | 26 |
| Total Accounted Records | 1,316 |

**Calculation:**

1,269 + 21 + 26 = 1,316

**Reconciliation Status: PASSED**

## 7. Expected Outcome

After applying these data-quality rules:

- Duplicate bookings are removed.
- Extra spaces are eliminated.
- Text values are standardized.
- Missing city and booking channel values are handled.
- Missing nights and total amounts are calculated.
- Missing guest IDs and ratings remain NULL.
- Invalid stay dates are rejected.
- Negative room rates are rejected.
- Cancelled bookings remain in Silver.
- Gold revenue and room-night calculations include only completed bookings.
- All source records are accounted for through reconciliation.

## 8. Conclusion

These data-quality rules help transform the raw hotel booking dataset into clean, consistent, and reliable data.

The Bronze layer preserves the original records, the Silver layer handles cleaning and validation, and the Gold layer provides trusted business summaries.

This approach improves data accuracy, supports reliable reporting, and makes the overall data engineering pipeline repeatable.
