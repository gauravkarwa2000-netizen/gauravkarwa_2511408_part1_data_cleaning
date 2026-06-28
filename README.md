# Part 1 — Business Data Cleaning, Validation & Excel Reporting

## Problem Summary

A retail company exports order-level sales data from multiple internal systems. The raw dataset (`raw_orders.xlsx`) contains 932 rows across 21 columns with numerous data quality issues: inconsistent formatting, multiple date formats, duplicate records, missing values, invalid discounts, and mixed order statuses. The goal is to produce a clean, validated, analysis-ready dataset along with summary reports for business review.

---

## Dataset Description

| Attribute | Detail |
|-----------|--------|
| File | `data/raw_orders.xlsx` |
| Sheet | `raw_orders` |
| Rows | 932 (including header) |
| Columns | 21 |
| Date range | 2024–2025 |
| Regions covered | North, South, East, West |
| Key fields | order_id, order_date, ship_date, customer, segment, region, category, sales, discount, profit, payment_status, order_status |

---

## Tools Used

- **Python 3.12** — data cleaning, analysis, file generation
- **pandas** — data manipulation and pivot summaries
- **openpyxl** — Excel file creation with formatting and formulas
- **dateutil** — robust date parsing across mixed formats
- **Microsoft Excel / LibreOffice Calc** — for review and screenshots

---

## Cleaning Steps Performed

1. **Preserved raw file** — `raw_orders.xlsx` kept unchanged; all work done in `cleaned_orders.xlsx`
2. **Text standardization** — Trimmed whitespace, collapsed multiple spaces, applied Title Case to all text fields. Fixed variants like `"NORTH"→"North"`, `"Small  Business"→"Small Business"`, `"storage"→"Storage"`
3. **Date parsing** — Parsed 7 different date formats (DD-MM-YYYY, MM/DD/YYYY, YYYY-MM-DD, DD Mon YYYY, etc.) and standardized to DD-MM-YYYY
4. **Date validation** — Identified ship dates earlier than order dates and flagged them
5. **Duplicate handling** — Removed 20 exact duplicate rows; flagged 12 conflicting duplicate order IDs for review
6. **Missing value treatment** — Filled missing `region` and `ship_mode` with "Unknown"; set missing `discount` to 0 where valid
7. **Discount validation** — Flagged 16 negative discounts and 2 discounts > 60% as invalid
8. **Calculated columns** — Added `cleaned_discount`, `calculated_sales`, `calculated_profit`, `profit_margin`, `shipping_delay_days`, `order_month`, `order_year`, `data_quality_flag`

---

## Business Rules Applied

| Rule | Action |
|------|--------|
| Missing region | Filled "Unknown", flagged |
| Missing ship_mode | Filled "Unknown", flagged |
| Missing discount | Set to 0 (if other fields valid) |
| Negative discount | Flagged Invalid |
| Discount > 60% | Flagged Invalid |
| Cancelled orders | Excluded from completed sales summary |
| Failed payments | Excluded from sales summary |
| Refunded orders | Summarized separately |
| Ship date < order date | Flagged Invalid |
| Exact duplicate rows | Removed (first kept) |
| Conflicting duplicate IDs | Flagged, not removed |

---

## Summary of Data Quality Issues Found

| Issue | Count |
|-------|-------|
| Exact duplicate rows removed | 20 |
| Missing region (filled Unknown) | 26 |
| Missing ship_mode (filled Unknown) | 22 |
| Missing discount (set to 0) | 26 |
| Negative discount | 16 |
| Discount > 60% | 2 |
| Ship date before order date | 5 |
| Conflicting duplicate order IDs | 12 |
| **Total Clean records** | **769** |
| **Total Warning records** | **86** |
| **Total Invalid records** | **57** |

---

## Summary of Final Pivot Reports (`pivot_summary.xlsx`)

| Sheet | Contents |
|-------|----------|
| `By Region` | Total Sales, Profit, Order Count, Profit Margin % by region — sorted by sales descending |
| `By Category` | Sales and Profit by Category × Sub-Category — sorted by sales |
| `By Ship Mode` | Order count breakdown by ship mode (all orders + completed/cancelled split) |
| `By Segment` | Profit margin % by customer segment — sorted by margin |
| `Cancelled-Refunded-Failed` | Cancelled/Returned orders by region; Failed/Refunded payments by region |
| `Monthly Sales Trend` | Sales, Profit, Order Count by Year × Month (completed paid orders only) |

---

## Key Business Insights

1. **West and South regions** generate the highest sales volume, with West leading in absolute revenue.
2. **Technology category** (Phones, Copiers, Accessories) drives the highest profit margins.
3. **Standard Class** is the most used shipping mode, while **Same Day** is least common.
4. **Corporate segment** shows consistently higher profit margins than Consumer.
5. **~17% of orders** are Cancelled, Returned, or involve Failed/Refunded payments — a significant leakage rate requiring operational review.
6. **Furniture sub-categories** (Tables, Bookcases) show lower profit margins relative to sales.
7. **Seasonal peaks** observed in Q4 (Oct–Dec) for both 2024 and 2025 data.

---

## Assumptions and Limitations

- Discount is assumed to be a decimal fraction (0.20 = 20% off)
- Ambiguous dates defaulted to day-first parsing
- `calculated_sales = quantity × unit_price × (1 - cleaned_discount)`
- Only `Completed` + `Paid` orders used in sales pivot summaries
- No external product catalog was available for price validation
- Conflicting duplicate order IDs require manual business review to resolve

---

## Screenshots Included

| File | Shows |
|------|-------|
| `screenshots/raw_data_preview.png` | Raw dataset before cleaning |
| `screenshots/cleaned_data_preview.png` | Cleaned dataset with all new columns |
| `screenshots/pivot_summary_1.png` | Sales & Profit by Region pivot |
| `screenshots/pivot_summary_2.png` | Monthly Sales Trend pivot |
