# Cleaning Log — raw_orders.xlsx

## Issues Found

| # | Issue | Count |
|---|-------|-------|
| 1 | Exact duplicate rows | 20 |
| 2 | Duplicate order IDs (conflicting data) | 32 |
| 3 | Missing region | 26 |
| 4 | Missing ship_mode | 22 |
| 5 | Missing discount | 18 |
| 6 | Negative discounts | 16 |
| 7 | Discounts in percentage format (e.g., "70%") | 8 |
| 8 | Excessive discounts (>70%) | Multiple |
| 9 | Ship date before order date | See report |
| 10 | Mixed date formats across columns | All rows |
| 11 | Inconsistent text casing (ALL CAPS, lowercase, title) | All text cols |
| 12 | Extra/leading/trailing whitespace in text fields | All text cols |
| 13 | Sales calculation mismatches | See report |

---

## Cleaning Actions Performed

### Text Fields
- Applied `.strip()` and `.replace(r'\s+', ' ')` to remove extra spaces from all text columns
- Normalized casing using `.str.title()` then mapped to canonical values
- Columns cleaned: `customer_name`, `segment`, `region`, `state`, `city`, `category`, `sub_category`, `ship_mode`, `payment_status`, `order_status`
- Handled all alias forms (e.g., "COMPLETED", "completed", "  Completed " → "Completed")

### Date Columns
- Parsed both `order_date` and `ship_date` using multiple format patterns:
  - `DD Mon YYYY` (e.g., "21 Jul 2024")
  - `MM/DD/YYYY` (e.g., "07/27/2024")
  - `YYYY-MM-DD` (e.g., "2024-01-22")
  - `DD-MM-YYYY` (e.g., "30-01-2024")
  - `DD-Mon-YYYY` (e.g., "30-Jan-2024")
- Created `shipping_delay_days` = `ship_date` − `order_date`
- Created `order_month` and `order_year` from `order_date`
- Flagged negative `shipping_delay_days` as "Invalid"

### Duplicates
- Removed 20 exact duplicate rows (all columns identical)
- Kept the first occurrence of duplicate `order_id` records; flagged subsequent ones as "Invalid"

### Missing Values
- **region**: Filled with `"Unknown"` (26 records) per business rule
- **ship_mode**: Filled with `"Unknown"` (22 records) per business rule
- **discount**: Treated as `0` where all other sales fields were valid (18 records)

### Discount
- Converted percentage strings (e.g., `"70%"`) to decimal fractions (e.g., `0.70`)
- Flagged negative discounts as "Invalid"
- Flagged discounts > 70% as "Warning" (unusual for retail)

### Calculated Columns Added
| Column | Formula |
|--------|---------|
| `cleaned_discount` | Normalized discount value |
| `calculated_sales` | `quantity × unit_price × (1 − cleaned_discount)` |
| `calculated_profit` | `calculated_sales − cost` |
| `profit_margin` | `calculated_profit / calculated_sales` |
| `shipping_delay_days` | `ship_date − order_date` (days) |
| `order_month` | Month number extracted from `order_date` |
| `order_year` | Year extracted from `order_date` |
| `data_quality_flag` | "Clean", "Warning", or "Invalid" |
| `flag_reason` | Semicolon-delimited list of issues |

---

## Business Rules Applied

| Rule | Action |
|------|--------|
| Missing `region` | Filled with "Unknown"; flagged in quality report |
| Missing `ship_mode` | Filled with "Unknown"; flagged in quality report |
| Missing `discount` | Treated as 0 if all other sales fields are valid |
| Negative `discount` | Flagged as Invalid |
| Discount > 70% | Flagged as Warning (excessive for retail) |
| Cancelled orders | Excluded from completed sales pivot summaries |
| Failed payments | Excluded from completed sales pivot summaries |
| Returned orders | Summarized in a dedicated pivot section |
| Ship date < Order date | Flagged as Invalid |
| Exact duplicate rows | Removed (first occurrence kept) |
| Conflicting duplicate `order_id` | First kept; rest flagged as Invalid |

---

## Assumptions Made

1. **Date format ambiguity**: Where date format was ambiguous (e.g., `06/08/2024` could be Jun-8 or Aug-6), the `MM/DD/YYYY` format was applied consistently based on the majority format pattern in the dataset.
2. **Discount threshold**: Discounts above 70% were treated as "Warning" rather than "Invalid" — they could be legitimate bulk/clearance discounts, but warrant review.
3. **Missing discount = 0**: Only applied where `quantity`, `unit_price`, `sales`, and `cost` fields were all present and non-zero.
4. **Sales mismatch tolerance**: A difference of ≤ $1 between `sales` and `calculated_sales` was considered acceptable (rounding).
5. **"Unknown" for missing region/ship_mode**: These were not removed; business may still need the order records for other reporting purposes.
6. **Refunded payment status**: Records with `payment_status = "Refunded"` were not removed; they are summarized separately and excluded from completed-sales analysis.

---

## Records Removed

- **20 exact duplicate rows** removed from the cleaned dataset.
- No other records were removed; all issues were handled by flagging.

## Records Flagged

- **Invalid**: Records with negative discounts, ship date before order date, missing critical dates, or conflicting duplicate order IDs.
- **Warning**: Records with excessive discounts, sales/calculation mismatches, or other anomalies needing review.

---

## Limitations

1. Date parsing may mis-classify ambiguous dates (e.g., `06/08/2024`).
2. The `sales` column in the raw data may reflect applied promotions or rounding not captured in `unit_price × quantity × (1-discount)` — these are flagged but not corrected.
3. `product_name` was not standardized (too many unique values; would require a product master reference).
4. Customer deduplication was not performed — the same customer could appear under slightly different names.
5. No currency normalization was applied (assumed all values are in the same currency, INR based on Indian cities/states).
