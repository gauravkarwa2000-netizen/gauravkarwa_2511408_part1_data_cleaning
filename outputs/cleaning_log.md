# Data Cleaning Log — raw_orders.xlsx

---

## 1. Issues Found

| # | Issue | Count | Location |
|---|-------|-------|----------|
| 1 | Exact duplicate rows | 20 | Multiple rows |
| 2 | Conflicting duplicate order_id values | 12 | e.g. ORD-2024-10124, ORD-2025-10007 |
| 3 | Missing `region` | 26 | Various rows |
| 4 | Missing `ship_mode` | 22 | Various rows |
| 5 | Missing `discount` (blank/null) | 26 | Various rows |
| 6 | Negative discount values | 16 | e.g. -0.19, -0.23, -0.14 |
| 7 | Discount > 60% | 2 | Unusually high (0.65) |
| 8 | Ship date earlier than order date | 5 | e.g. ORD-2024-10143, ORD-2025-10128 |
| 9 | Inconsistent text case (region, segment, ship_mode, sub_category) | ~40 | Mixed UPPER/lower/Title |
| 10 | Multiple spaces in text fields | ~5 | e.g. "Small  Business", "Standard  Class" |
| 11 | Inconsistent date formats | All | Mix of DD-MM-YYYY, MM/DD/YYYY, YYYY-MM-DD, DD Mon YYYY |
| 12 | Cancelled/Returned/Failed orders mixed into dataset | ~160 | Various |

---

## 2. Cleaning Actions Performed

### Text Fields
- Applied `TRIM` equivalent (strip + collapse multiple spaces) to: `customer_name`, `segment`, `region`, `state`, `city`, `category`, `sub_category`, `ship_mode`, `payment_status`, `order_status`
- Converted all text values to **Title Case** using Python `.title()`
- Manually mapped variants: `"Small  Business"` → `"Small Business"`, `"Standard  Class"` → `"Standard Class"`, `"storage"` → `"Storage"`, `"NORTH"` → `"North"`, `"FIRST CLASS"` → `"First Class"`

### Dates
- Parsed all dates from mixed formats (7 format patterns tried per cell)
- Standardized output to `DD-MM-YYYY` in `cleaned_orders.xlsx`
- Created `shipping_delay_days` = `ship_date - order_date` (days)
- Created `order_month` and `order_year` from `order_date`

### Duplicates
- **Exact duplicates** (all 21 columns identical): 20 rows removed, keeping first occurrence
- **Conflicting duplicates** (same `order_id`, different data): flagged with `data_quality_flag = Warning` or `Invalid`; NOT removed silently

### Missing Values
- `region`: 26 missing → filled as **"Unknown"**, flagged in quality report
- `ship_mode`: 22 missing → filled as **"Unknown"**, flagged in quality report
- `discount`: 26 blank/null → set to **0** (where all other sales fields valid), flagged

### Discount Validation
- Negative discounts (16 rows): **flagged as Invalid** — discount cannot be negative
- Discount > 0.6 (2 rows): **flagged as Invalid** — unusually high, likely data entry error

---

## 3. Business Rules Applied

| Rule | Action |
|------|--------|
| Missing region | Filled as "Unknown"; flagged in quality report |
| Missing ship_mode | Filled as "Unknown"; flagged in quality report |
| Missing discount | Treated as 0 where other sales fields are valid |
| Negative discount | Flagged as Invalid in `data_quality_flag` column |
| Discount > 60% | Flagged as Invalid |
| Cancelled orders | Excluded from completed sales summary in pivot reports |
| Failed payments | Excluded from completed sales summary |
| Refunded orders | Summarized separately in `Cancelled-Refunded-Failed` pivot sheet |
| Ship date before order date | Flagged as Invalid shipping record |
| Exact duplicate rows | Removed (first instance kept) |
| Conflicting duplicate order IDs | Flagged for review, NOT removed |

---

## 4. Assumptions Made

1. **Discount as proportion**: All discount values treated as decimal fractions (e.g., 0.20 = 20% off), consistent with formula `sales = quantity × unit_price × (1 - discount)`.
2. **Date format ambiguity**: For dates like `03/01/2024`, used context clues and surrounding data to determine month-first vs day-first. Defaulted to `dayfirst=True` when ambiguous.
3. **Missing discount = 0**: When discount is blank and the row otherwise has valid quantity, price, and sales figures, discount is set to 0 for `cleaned_discount`.
4. **"Completed" + "Paid" = valid sales**: Pivot summaries for sales/profit include only rows where `order_status == "Completed"` AND `payment_status == "Paid"`.
5. **Refunded = separate category**: Refunded orders are neither included in completed sales nor treated as failed — they have their own summary section.
6. **Calculated sales formula**: `calculated_sales = quantity × unit_price × (1 - cleaned_discount)`. Where `cleaned_discount` is negative or > 0.6, the original invalid value is used for the flag but `calculated_sales` uses `max(0, cleaned_discount)` to avoid negative sales.
7. **Profit**: `calculated_profit = calculated_sales - cost`.
8. **Profit margin**: `profit_margin = calculated_profit / calculated_sales` (blank if sales = 0).

---

## 5. Records Removed

| Reason | Count |
|--------|-------|
| Exact duplicate rows | 20 |
| **Total removed** | **20** |

Remaining records: **912**

---

## 6. Records Flagged

| Flag | Count | Reason |
|------|-------|--------|
| Invalid | 57 | Negative discount, ship-before-order date, exact duplicates (before removal) |
| Warning | 86 | Missing region/ship_mode (now filled), missing discount (now 0), conflicting duplicate IDs |
| Clean | 769 | No issues detected |

---

## 7. Limitations

1. **Date parsing**: Dates in `DD Mon YYYY` format (e.g., "21 Jul 2024") parsed successfully. However, ambiguous dates like `03/01/2024` may have been misinterpreted as March 1 vs January 3 in a small number of cases.
2. **Calculated sales vs. original sales**: Differences exist in ~30+ rows — this may reflect rounding in the source system, bundle pricing, or other discounts not captured in the `discount` column.
3. **Product-level validation**: No product catalog was available to validate `product_name` or `unit_price` ranges.
4. **Customer deduplication**: Customer IDs and names not cross-validated — same customer may appear under slightly different spellings.
5. **No geographic validation**: State/city combinations were not validated against an authoritative list of Indian states/cities.
6. **Date `02/29/2024`**: 2024 is a leap year so Feb 29 is valid. Retained as-is.
7. **Conflicting duplicates**: 12 rows with same `order_id` but different data are flagged but not resolved — business decision required.
