# GauravKarwa_2511408_Part1_DataCleaning
# Assumptions Made During Data Cleaning

## Date Format Assumptions

1. The dataset contained multiple date formats including:

   * DD MMM YYYY, MM/DD/YYYY, DD-MM-YYYY, YYYY-MM-DD

2. Slash-formatted dates were interpreted as MM/DD/YYYY because several records contained dates such as 08/31/2024 and 07/27/2024, which cannot be valid DD/MM/YYYY dates.

3. All valid dates were standardized to a single display format:
   DD-MMM-YYYY.

4. Records with ship_date earlier than order_date were not corrected automatically and were flagged as invalid shipping records.

---

## Discount Assumptions

5. The assignment did not define the maximum allowable discount percentage. A maximum valid discount of 50% was assumed.

6. Any discount value greater than 50% was flagged as an invalid discount.

7. Negative discount values were considered invalid and were flagged for review.

8. Missing discount values were replaced with 0 only when quantity and unit price fields were available and valid, as required by the business rules.

---

## Duplicate Handling Assumptions

9. Exact duplicate rows were treated as system-generated duplicates and were removed from analytical reporting.

10. Duplicate order IDs with conflicting information were retained and flagged for manual review rather than deleted.

11. Duplicate order IDs were assumed to represent potential data integration or transactional issues and therefore required investigation.

---

## Missing Value Assumptions

12. Missing region values were replaced with "Unknown" as specified in the business rules.

13. Missing ship_mode values were replaced with "Unknown" as specified in the business rules.

14. Other missing values were retained unless a specific business rule required replacement.

---

## Sales and Profit Validation Assumptions

15. Calculated sales were assumed to follow the formula:

Sales = Quantity × Unit Price × (1 − Discount)

16. Calculated profit was assumed to follow the formula:

Profit = Calculated Sales − Cost

17. Minor calculation differences caused by decimal rounding were tolerated up to ±0.01.

18. Any difference greater than ±0.01 between reported and calculated values was flagged as a mismatch.

---

## Reporting Assumptions

19. Cancelled orders were excluded from completed sales analysis.

20. Failed payment orders were excluded from completed sales analysis.

21. Refunded orders were summarized separately and not included in completed sales reporting.

22. Monthly trend analysis was based on the standardized order_date field.

23. Profit margin was calculated as:

Profit Margin = Profit ÷ Sales

24. Records flagged as WARNING remained available for analysis but were highlighted in data quality reporting.

25. Records flagged as INVALID were retained in the dataset for auditability and transparency but were separately identified in data quality reports.
