# Part 1: Data Cleaning & Analysis — Retail Orders Dataset

## Problem Summary

The retail company exported order-level sales data from multiple internal systems into `raw_orders.xlsx`. The dataset contained numerous data quality issues including inconsistent text formatting, mixed date formats, duplicate records, missing values, invalid discounts, calculation mismatches, and order status inconsistencies. This project cleans, validates, and prepares the data for business analysis.

---

## Dataset Description

| Attribute | Detail |
|-----------|--------|
| File | `raw_orders.xlsx` |
| Original Records | 932 rows |
| Columns | 21 (order_id, dates, customer info, product info, financials, status) |
| Date Range | 2024–2025 |
| Geography | India (states and cities) |

**Columns:** `order_id`, `order_date`, `ship_date`, `customer_id`, `customer_name`, `segment`, `region`, `state`, `city`, `category`, `sub_category`, `product_name`, `ship_mode`, `quantity`, `unit_price`, `discount`, `sales`, `cost`, `profit`, `payment_status`, `order_status`

---

## Tools Used

- **Python 3.12** — all data cleaning and report generation
- **pandas** — data manipulation, aggregation, pivot analysis
- **openpyxl** — Excel file creation with formatting
- **matplotlib** — chart and screenshot generation

---

## Cleaning Steps Performed

1. **Preserved original file** — all cleaning done in a separate `cleaned_orders.xlsx`
2. **Text standardization** — stripped whitespace, normalized casing, resolved all aliases across 10 text columns
3. **Date parsing** — handled 5+ mixed date formats; standardized to consistent datetime
4. **Duplicate handling** — removed 20 exact duplicates; flagged 32 conflicting duplicate order IDs
5. **Missing values** — filled `region` and `ship_mode` with "Unknown"; treated missing `discount` as 0
6. **Discount cleaning** — converted % strings to decimals; flagged negative and excessive discounts
7. **Calculated columns** — added 8 derived columns including `calculated_sales`, `profit_margin`, `shipping_delay_days`
8. **Data quality flagging** — assigned each record a flag: `Clean`, `Warning`, or `Invalid`

---

## Business Rules Applied

| Rule | Action |
|------|--------|
| Missing region | Filled "Unknown" |
| Missing ship_mode | Filled "Unknown" |
| Missing discount | Treated as 0 |
| Negative discount | Flagged Invalid |
| Discount > 70% | Flagged Warning |
| Cancelled orders | Excluded from sales summary |
| Failed payments | Excluded from sales summary |
| Returned orders | Separately summarized |
| Ship date before order date | Flagged Invalid |

---

## Summary of Data Quality Issues

| Issue | Count |
|-------|-------|
| Exact duplicate rows removed | 20 |
| Duplicate order IDs (conflicting) | 32 |
| Missing region | 26 |
| Missing ship_mode | 22 |
| Missing discount | 18 |
| Negative discounts | 16 |
| Percentage-format discounts | 8 |
| Ship date before order date | (see report) |
| Sales calculation mismatches | (see report) |
| **Total Clean records** | **829** |
| **Total Warning records** | **36** |
| **Total Invalid records** | **47** |

---

## Summary of Pivot Reports

### `pivot_summary.xlsx` contains 6 sheets:

1. **Region Sales & Profit** — Total sales, profit, and average margin by region (sorted by sales)
2. **Category Sub-Cat** — Breakdown by category and sub-category
3. **Ship Mode Orders** — Order volume by shipping method
4. **Profit Margin by Segment** — Margin analysis by customer segment
5. **Issues by Region** — Cancelled, returned, failed, and refunded orders by region
6. **Monthly Sales Trend** — Month-by-month revenue and profit for completed orders

---

## Key Business Insights

- **North and South regions** generate the highest completed sales volume
- **Technology** category yields the best profit margins despite lower volume than Office Supplies
- **Standard Class** shipping is the most common mode; **Same Day** has the fewest orders
- **Consumer** segment dominates order volume; **Corporate** tends to have better margins
- Significant spike in orders in mid-2025 based on monthly trend
- **16 records with negative discounts** — likely a data entry error in the source system
- **32 duplicate order IDs with conflicting data** indicate issues at the source system integration layer

---

## Assumptions and Limitations

1. Currency assumed to be INR (Indian Rupees) based on geography
2. Ambiguous dates (e.g., `06/08/2024`) defaulted to `MM/DD/YYYY`
3. Discount > 70% treated as Warning, not Invalid (could be legitimate clearance)
4. No customer name deduplication performed
5. `product_name` not standardized (no product master available)
6. Sales mismatch records flagged but original `sales` value retained

---

## Screenshots

| File | Description |
|------|-------------|
| `raw_data_preview.png` | First 8 rows of raw dataset before cleaning |
| `cleaned_data_preview.png` | Cleaned dataset with calculated columns and quality flags |
| `pivot_summary_1.png` | Sales & Profit by Region |
| `pivot_summary_2.png` | Monthly Sales & Profit |
