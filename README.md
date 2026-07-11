# sales-order-eda

Exploratory Data Analysis (EDA) on a 1,200-row e-commerce sales order dataset — built as Project 2 of the DecodeLabs Data Analytics Internship (2026 batch).

---

## Table of Contents

1. [Problem Statement](#problem-statement)
2. [Dataset Overview](#dataset-overview)
3. [Methodology](#methodology)
4. [Data Quality & Cleaning](#data-quality--cleaning)
5. [Descriptive Statistics](#descriptive-statistics)
6. [Distribution Analysis](#distribution-analysis)
7. [Outlier Detection](#outlier-detection)
8. [Categorical Analysis](#categorical-analysis)
9. [Time-Based Trends](#time-based-trends)
10. [Correlation Analysis](#correlation-analysis)
11. [Business Breakdown by Product & Payment Method](#business-breakdown-by-product--payment-method)
12. [Key Findings](#key-findings)
13. [Recommendations](#recommendations)
14. [Limitations](#limitations)
15. [Repository Structure](#repository-structure)
16. [How to Run](#how-to-run)
17. [Tools Used](#tools-used)

---

## Problem Statement

Before any dashboard or predictive model can be trusted, the underlying data has to be understood: what shape is it, where are the gaps, and what patterns already exist? This project analyzes a raw sales order dataset to answer four questions:

1. What do the core business metrics (order size, price, revenue) actually look like — and are the averages we'd normally report even trustworthy?
2. Where are the outliers and data quality issues, and are they errors or real signal?
3. What patterns exist across products, payment methods, referral sources, and time that could inform business decisions?
4. Is there a business-critical problem hiding in the data that a surface-level report would miss?

## Dataset Overview

- **File:** `sales_data.csv`
- **Rows:** 1,200 orders
- **Columns:** 14
- **Fields:**
  - `OrderID` — unique order identifier (1,200 unique values, no duplicates)
  - `Date` — order date (671 unique dates in the range)
  - `CustomerID` — customer identifier (1,189 unique customers across 1,200 orders)
  - `Product` — one of 7 categories: Chair, Printer, Laptop, Tablet, Monitor, Desk, Phone
  - `Quantity` — units purchased per order
  - `UnitPrice` — price per unit ($)
  - `ShippingAddress` — delivery address (655 unique values)
  - `PaymentMethod` — one of 5 methods: Online, Cash, Credit Card, Debit Card, Gift Card
  - `OrderStatus` — one of 5 states: Cancelled, Returned, Pending, Shipped, Delivered
  - `TrackingNumber` — shipment tracking ID
  - `ItemsInCart` — number of items in the cart at checkout
  - `CouponCode` — one of 3 codes (FREESHIP, WINTER15, SAVE10) or blank if none used
  - `ReferralSource` — one of 5 channels: Instagram, Email, Google, Facebook, Referral
  - `TotalPrice` — final order value ($)
- **Date range:** 2023-01-01 to 2025-06-30
- **Total revenue across all orders:** $1,264,761.96 (sum of all seven product-level totals)

## Methodology

The notebook (`Sales_Order_EDA.ipynb`) follows this pipeline:

1. **Import libraries** — pandas, NumPy, Matplotlib, Seaborn.
2. **Load the dataset** and confirm shape (1,200 rows × 14 columns).
3. **Initial inspection** — `df.info()` and `df.describe(include='all')` to check data types and get first-look statistics.
4. **Data quality audit** — check every column for missing values, and check for duplicate rows and duplicate `OrderID`s.
5. **Cleaning** — resolve the one column with missing data (`CouponCode`) based on business logic; convert `Date` to datetime and derive `Year`, `Month`, `MonthName`, and `DayOfWeek`.
6. **Descriptive statistics** — count, mean, median, standard deviation, min, max for all four numeric columns.
7. **Skew check** — compare mean vs. median for each numeric column to flag skewed distributions.
8. **Distribution analysis** — histograms with mean/median overlays for all four numeric columns.
9. **Boxplots** — visualize spread and flag potential outliers for all four numeric columns.
10. **Outlier detection** — IQR method (`Q1 − 1.5×IQR` to `Q3 + 1.5×IQR`) applied to each numeric column, with the flagged `TotalPrice` rows inspected directly.
11. **Categorical analysis** — count plots and percentage breakdowns for `Product`, `PaymentMethod`, `OrderStatus`, `ReferralSource`, `CouponCode`.
12. **Time-based trends** — monthly order count/revenue line charts, and order volume by day of week.
13. **Correlation analysis** — Pearson correlation matrix and heatmap across the four numeric columns.
14. **Business breakdown** — revenue/order count by `Product`, by `OrderStatus`, and by `PaymentMethod`.
15. **Key findings printout** — a final code cell that programmatically prints the headline conclusions.

## Data Quality & Cleaning

- **Missing values:** Only one column had nulls — `CouponCode`, with 309 missing values (25.75% of rows). All other 13 columns are 100% non-null.
- **Interpretation:** A missing `CouponCode` means the customer checked out without applying a coupon, not that data was lost or corrupted.
- **Action taken:** Nulls in `CouponCode` were relabeled `"No Coupon"`, preserving all 1,200 rows. Post-cleaning breakdown: FREESHIP 313, No Coupon 309, WINTER15 292, SAVE10 286.
- **Duplicates:** 0 duplicate rows and 0 duplicate `OrderID`s were found.
- **Type correction:** `Date` was converted from an object/string type to `datetime64`, enabling `Year`, `Month`, `MonthName`, and `DayOfWeek` to be derived.

**Result:** after cleaning, the dataset has 0% missing values and 0% duplicate records.

## Descriptive Statistics

| Metric | Quantity | UnitPrice ($) | ItemsInCart | TotalPrice ($) |
|---|---|---|---|---|
| Count | 1,200 | 1,200 | 1,200 | 1,200 |
| Mean | 2.95 | 356.41 | 5.48 | 1,053.97 |
| Median | 3.00 | 364.21 | 5.00 | 823.62 |
| Std Dev | 1.41 | 197.18 | 2.28 | 819.86 |
| Min | 1.00 | 11.39 | 1.00 | 11.39 |
| 25th percentile (Q1) | 2.00 | 186.06 | 4.00 | 410.52 |
| 75th percentile (Q3) | 4.00 | 521.57 | 7.00 | 1,578.48 |
| Max | 5.00 | 699.93 | 10.00 | 3,456.40 |

**Mean vs. median skew check (from the notebook's own output):**

```
Quantity     | Mean:      2.95 | Median:      3.00 | Roughly symmetric
UnitPrice    | Mean:    356.41 | Median:    364.21 | Roughly symmetric
ItemsInCart  | Mean:      5.49 | Median:      5.00 | Right-skewed (mean > median)
TotalPrice   | Mean:   1053.97 | Median:    823.62 | Right-skewed (mean > median)
```

## Distribution Analysis

Histograms with mean/median overlays were plotted for all four numeric variables. Consistent with the skew check above, `TotalPrice` and `ItemsInCart` show a longer tail toward higher values, while `Quantity` and `UnitPrice` show more even, symmetric spread.

## Outlier Detection

Using the IQR method (values below `Q1 − 1.5×IQR` or above `Q3 + 1.5×IQR` flagged as outliers), the notebook's own output is:

```
Quantity     | Valid range: [   -1.00,     7.00] | Outliers found: 0
UnitPrice    | Valid range: [ -317.20,  1024.83] | Outliers found: 0
ItemsInCart  | Valid range: [   -0.50,    11.50] | Outliers found: 0
TotalPrice   | Valid range: [-1341.41,  3330.41] | Outliers found: 8
```

The 8 flagged `TotalPrice` orders were inspected directly. The highest is **$3,456.40** (Order `ORD200789`, Tablet, 5 units, Online, Delivered), and the lowest of the eight is **$3,334.00** (Order `ORD201065`, Printer, 5 units, Debit Card, Delivered). All 8 outlier orders share `Quantity = 5` (the dataset maximum) combined with a high `UnitPrice`, confirming these are large-but-legitimate combinations rather than an error in any single field.

## Categorical Analysis

**Product mix** (share of 1,200 orders):

| Product | Share |
|---|---|
| Printer | 15.1% |
| Tablet | 14.9% |
| Chair | 14.8% |
| Laptop | 14.4% |
| Desk | 14.2% |
| Monitor | 13.6% |
| Phone | 13.0% |

**Payment method mix:**

| Method | Share |
|---|---|
| Online | 21.5% |
| Cash | 20.5% |
| Credit Card | 19.5% |
| Debit Card | 19.3% |
| Gift Card | 19.2% |

**Order status mix:**

| Status | Share |
|---|---|
| Cancelled | 20.8% |
| Returned | 20.6% |
| Pending | 19.8% |
| Shipped | 19.6% |
| Delivered | 19.2% |

**Referral source mix:**

| Source | Share |
|---|---|
| Instagram | 21.6% |
| Email | 20.8% |
| Google | 20.1% |
| Facebook | 19.0% |
| Referral | 18.5% |

**Coupon usage mix** (post-cleaning, includes "No Coupon"):

| Code | Share |
|---|---|
| FREESHIP | 26.1% |
| No Coupon | 25.8% |
| WINTER15 | 24.3% |
| SAVE10 | 23.8% |

All five categorical fields are notably evenly distributed — no single category dominates any dimension by a wide margin.

## Time-Based Trends

The notebook plots monthly order count/revenue and order volume by day of week. Aggregating the same underlying data confirms:

- **Data spans 30 distinct calendar months**, from January 2023 through June 2025.
- **Busiest month by order count:** June 2024, with 53 orders ($68,068.54 in revenue — the highest of any month).
- **Quietest month by order count:** January 2025, with 27 orders.
- **Lowest revenue month:** April 2023, at $27,751.71.
- Order count and revenue move together month-to-month with no sustained multi-year upward or downward trend — the business appears roughly stable across the 30-month window rather than growing or declining.

**Day-of-week order distribution (1,200 orders total):**

| Day | Orders |
|---|---|
| Monday | 174 |
| Tuesday | 165 |
| Wednesday | 163 |
| Thursday | 166 |
| Friday | 174 |
| Saturday | 172 |
| Sunday | **186** |

Sunday has the highest order volume (186 orders, ~15.5%); Wednesday is the lowest (163 orders, ~13.6%).

## Correlation Analysis

Pearson correlation matrix, from the notebook's own output:

| | Quantity | UnitPrice | ItemsInCart | TotalPrice |
|---|---|---|---|---|
| **Quantity** | 1.000 | 0.015 | 0.650 | 0.615 |
| **UnitPrice** | 0.015 | 1.000 | 0.001 | 0.717 |
| **ItemsInCart** | 0.650 | 0.001 | 1.000 | 0.393 |
| **TotalPrice** | 0.615 | 0.717 | 0.393 | 1.000 |

**Interpretation:**

- `UnitPrice` has the strongest relationship with `TotalPrice` (r = 0.717), followed by `Quantity` (r = 0.615) — expected, since `TotalPrice` is derived from these two fields.
- `ItemsInCart` correlates moderately with `Quantity` (r = 0.650) but only weakly with `TotalPrice` (r = 0.393), suggesting cart size is a soft proxy for quantity purchased, not a strong independent revenue driver.
- `Quantity` and `UnitPrice` are essentially uncorrelated (r = 0.015) — customers buying more units aren't systematically buying cheaper or pricier items.
- As the notebook's own commentary notes: **correlation is a clue, not a verdict.**

## Business Breakdown by Product & Payment Method

**Revenue by product** (from the notebook's `revenue_by_product` table):

| Product | Total Revenue | Avg Order Value | Order Count | Revenue Share |
|---|---|---|---|---|
| Chair | $195,620.11 | $1,098.99 | 178 | 15.5% |
| Printer | $195,612.61 | $1,080.73 | 181 | 15.5% |
| Laptop | $192,126.56 | $1,110.56 | 173 | 15.2% |
| Tablet | $186,568.95 | $1,042.28 | 179 | 14.8% |
| Monitor | $175,651.41 | $1,077.62 | 163 | 13.9% |
| Desk | $167,459.93 | $985.06 | 170 | 13.2% |
| Phone | $151,722.39 | $972.58 | 156 | 12.0% |

**Order status breakdown** (from the notebook's `status_summary` table):

| Status | Order Count | Total Value | Share |
|---|---|---|---|
| Cancelled | 250 | $276,396.21 | 20.8% |
| Returned | 247 | $243,277.70 | 20.6% |
| Pending | 237 | $256,328.15 | 19.8% |
| Shipped | 235 | $246,159.58 | 19.6% |
| Delivered | 231 | $242,600.32 | 19.2% |

**Revenue by payment method** (from the notebook's `payment_summary` table):

| Method | Order Count | Avg Order Value | Total Revenue |
|---|---|---|---|
| Credit Card | 234 | $1,127.55 | $263,847.63 |
| Online | 258 | $1,017.22 | $262,442.94 |
| Cash | 246 | $1,056.04 | $259,786.29 |
| Gift Card | 230 | $1,070.97 | $246,323.92 |
| Debit Card | 232 | $1,001.56 | $232,361.18 |

## Key Findings

Printed directly by the notebook's final code cell:

```
1. Dataset covers 2023-01-01 to 2025-06-30 (1200 orders).
2. Top revenue-generating product: Chair (15.5% of total revenue).
3. TotalPrice distribution is right-skewed -> median is a safer 'typical order value' than mean.
4. Combined Cancelled + Returned orders: 41.4% of all orders.
5. Outliers detected in TotalPrice: 8 orders (likely large legitimate bulk purchases, not necessarily errors — worth a manual review).
6. No missing data issues remain after treating CouponCode nulls as 'No Coupon' (0% of dataset had system errors).
```

Additional findings supported by the notebook's tables but not in the printed summary:

- Chair and Printer are effectively **tied** for top revenue ($195,620.11 vs. $195,612.61 — a difference of only $7.50), both at 15.5% share. The printed "top product" (Chair) technically edges out Printer only due to rounding at the second decimal.
- Product revenue share ranges only from 12.0% (Phone) to 15.5% (Chair/Printer) — a spread of 3.5 percentage points, indicating a balanced catalog rather than a single dominant product.
- Payment method revenue is tightly clustered ($232K–$264K across all 5 methods) — no method dominates.
- Credit Card has the highest average order value ($1,127.55) despite not having the highest order count, while Online has the highest order count (258) but the lowest average order value ($1,017.22).

## Recommendations

- **Report the median order value ($823.62), not the mean ($1,053.97),** in dashboards or stakeholder summaries — the mean overstates typical customer spend due to right-skew in `TotalPrice`.
- **Treat the 41.4% combined Cancelled/Returned rate as the top investigation priority.** This EDA identifies the pattern but does not diagnose its cause; a focused root-cause analysis (e.g., by product, shipping region, or order timing) is the logical next project.
- **Manually audit the 8 high-value `TotalPrice` outlier orders** — all involve the maximum `Quantity` (5) paired with high `UnitPrice`, so they are plausibly legitimate bulk-style purchases, but should be confirmed rather than assumed.
- **Don't position Chair as "the" top product over Printer** — the two are statistically tied ($7.50 apart in total revenue); marketing or inventory decisions should treat them as co-leaders, not rank one meaningfully above the other.
- **Don't over-invest in a single product line generally** — order share across all 7 products only ranges from 12.0% to 15.5%.
- **Investigate why Credit Card orders carry the highest average value** ($1,127.55) while Online drives the highest volume but lowest average value ($1,017.22) — this could inform which channel to prioritize for upsell versus volume campaigns.

## Limitations

- This is a **descriptive** EDA, not a causal analysis — the correlations reported here do not establish why cancellations or returns occur.
- The dataset does not include a "reason for cancellation/return" field, so the 41.4% combined rate can be quantified but not explained from this data alone.
- `ShippingAddress` values are highly repetitive (655 unique values across 1,200 rows, with some repeating up to 8 times), which limits how much can be concluded from geographic analysis; no location-based claims are made in this README as a result.
- Outlier flags (IQR method) identify statistical extremity, not confirmed errors — the 8 flagged `TotalPrice` orders require manual business context to classify as legitimate or erroneous.
- The notebook loads the CSV from a local file path (`C:\Users\SAIM\Downloads\sales_data.csv`); this must be updated to a relative path (e.g., `sales_data.csv`) before the notebook will run in a cloned repository — see [How to Run](#how-to-run).


```



**Before running:** update the file path in the second code cell from the original local path to a relative path so it works in any cloned copy of this repo:

```python
df = pd.read_csv("sales_data.csv")
```

`sales_data.csv` must be placed in the same folder as the notebook. Run all cells top to bottom.

## Tools Used

Python, pandas, NumPy, Matplotlib, Seaborn, Jupyter Notebook.

---
*Project 2 — Exploratory Data Analysis — DecodeLabs Data Analytics Internship, 2026 Batch.*
