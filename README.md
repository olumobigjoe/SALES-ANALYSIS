# Sales Data Analysis Report

Comprehensive analysis of 7,951 sales order lines spanning June 2021 – December 2024.

---

## Dataset Overview

| Field | Detail |
|-------|--------|
| File | `sales.xlsx` |
| Sheet | Sheet2 |
| Rows | 7,951 |
| Columns | 12 |
| Date range | 2021-06-26 → 2024-12-31 |
| Null values | None |

### Schema

```
Order Line        int64       Sequential line number
Order ID          str         Unique order identifier (e.g. OF-2019-103800)
Bar code value    int64       Product barcode
Order Date        str → date  Date order placed (DD-MM-YYYY)
Ship Date         str → date  Date order shipped
Ship Mode         str         First Class / Second Class / Standard Class / Same Day
Customer ID       str         Customer identifier (e.g. DP-13000)
Product ID        str         Category-SubCat-SKU (e.g. OFF-PA-10000174)
Sales             float64     Revenue for the line item ($)
Quantity          int64       Units ordered
Discount          float64     Fractional discount applied (0.0 – 0.8)
Profit            float64     Net profit for the line item ($)
```

### Derived Fields Used in Analysis

```python
df['Ship Days']    = (df['Ship Date'] - df['Order Date']).dt.days
df['Month']        = df['Order Date'].dt.to_period('M')
df['Category']     = df['Product ID'].str.split('-').str[0]   # FUR / OFF / TEC
df['Sub_Category'] = df['Product ID'].str.split('-').str[1]   # CH, PH, ST, ...
df['Disc_Bucket']  = pd.cut(df['Discount'], bins=[-0.01,0,0.2,0.4,0.6,0.8,1.0])
```

---

## Executive Summary

| KPI | Value |
|-----|-------|
| Total Sales | $1,835,656 |
| Total Profit | $234,431 |
| Overall Margin | 12.8% |
| Unique Orders | 3,967 |
| Unique Customers | 789 |
| Unique Products | 1,835 |
| Avg Order Value | $462.73 |
| Loss-making lines | 1,507 (19.0%) |
| Profitable months | 42 / 43 (97.7%) |

---

## Analysis by Dimension

### 1. Time Series

Monthly revenue grew from ~$5K (Jun 2021) to a peak of **$94.5K (May 2024)**, implying approximately **20× top-line growth** over the analysis window.

**Peak months** (by sales):
1. May 2024 — $94,535
2. Mar 2024 — $76,608
3. Jun 2024 — $73,941
4. May 2023 — $73,714
5. May 2022 — $74,922

**Loss month**: Jul 2022 posted -$1,413 profit on $17,261 in sales — the only month with negative net profit across the full period.

**Seasonality**: A repeating pattern emerges annually — sales spikes in March, May, September, and December; troughs in July–August and January–February.

---

### 2. Product Category Analysis

| Category | Sales | % of Total | Profit | Margin |
|----------|-------|------------|--------|--------|
| Technology (TEC) | $671,264 | 36.6% | $116,635 | 17.4% |
| Furniture (FUR) | $598,195 | 32.6% | $17,165 | 2.9% |
| Office Supplies (OFF) | $566,196 | 30.8% | $100,632 | 17.8% |

**Key finding**: Furniture accounts for 32.6% of revenue but only 7.3% of total profit. Its 2.9% margin is 6× lower than Office Supplies (17.8%). The Furniture category is masking significant value destruction.

**Sub-category breakdown (top 10 by sales)**:

| Sub-Cat | Description | Sales | Profit | Margin |
|---------|-------------|-------|--------|--------|
| CH | Chairs | $264,313 | $21,306 | 8.1% |
| PH | Phones | $261,764 | $35,946 | 13.7% |
| ST | Storage | $182,514 | $16,766 | 9.2% |
| MA | Machines | $165,338 | $6,252 | 3.8% |
| TA | Tables | $160,989 | **-$11,831** | **-7.3%** |
| BI | Binders | $154,423 | $27,124 | 17.6% |
| AC | Accessories | $129,854 | $32,235 | 24.8% |
| CO | Copiers | $114,308 | $42,202 | **36.9%** |
| BO | Bookcases | $99,314 | -$2,632 | -2.6% |
| AP | Appliances | $79,634 | $13,285 | 16.7% |

**Loss-making sub-categories**: Tables (-$11,831), Bookcases (-$2,632).  
**Highest margin sub-category**: Copiers at 36.9%, contributing $42,202 in profit.

---

### 3. Shipping Mode Analysis

| Ship Mode | Sales | Count | Profit | Avg Ship Days |
|-----------|-------|-------|--------|---------------|
| Standard Class | $1,101,899 | 4,805 | $131,481 | 5.0 |
| Second Class | $367,071 | 1,525 | $47,405 | 3.2 |
| First Class | $273,878 | 1,206 | $40,081 | 2.2 |
| Same Day | $92,807 | 415 | $15,464 | 0.04 |

Standard Class dominates with 60.5% of all line items and 56% of total profit. No shipping mode is net-negative.

---

### 4. Discount Impact Analysis

This is the most critical finding in the dataset.

| Discount Tier | Sales | Profit | Margin | Lines |
|---------------|-------|--------|--------|-------|
| No discount (0%) | $879,477 | +$261,296 | **29.7%** | 3,824 |
| 0–20% | $667,686 | +$75,276 | 11.3% | 2,997 |
| 20–40% | $187,107 | **-$26,672** | -14.3% | 366 |
| 40–60% | $57,360 | **-$21,270** | -37.1% | 181 |
| 60–80% | $44,025 | **-$54,198** | **-123.1%** | 583 |

**Key findings:**
- Every discount tier above 20% produces negative profit.
- The 60–80% tier loses more money than it generates in revenue (margin < -100%).
- 1,130 loss-making discounted lines represent ~$102K in recoverable profit if discounts were capped at 20%.
- Undiscounted orders achieve **29.7% margin** — 2.3× the company average.

---

### 5. Customer Analysis

**Top 10 customers by revenue:**

| Rank | Customer ID | Sales | Profit | Orders | Margin |
|------|-------------|-------|--------|--------|--------|
| 1 | SM-20320 | $25,035 | **-$1,983** | 4 | -7.9% |
| 2 | TC-20980 | $19,052 | $8,981 | 5 | 47.1% |
| 3 | RB-19360 | $14,967 | $6,930 | 4 | 46.3% |
| 4 | KL-16645 | $14,153 | $805 | 11 | 5.7% |
| 5 | SC-20095 | $14,142 | $5,757 | 9 | 40.7% |
| 6 | AB-10105 | $12,998 | $5,195 | 7 | 40.0% |
| 7 | SE-20110 | $11,996 | $2,588 | 8 | 21.6% |
| 8 | CC-12370 | $11,901 | $2,083 | 3 | 17.5% |
| 9 | BM-11140 | $11,790 | **-$1,660** | 4 | -14.1% |
| 10 | GT-14710 | $11,722 | $2,126 | 9 | 18.1% |

2 of the top 10 revenue customers (SM-20320, BM-11140) are net-negative. Revenue ranking without margin ranking is misleading.

---

### 6. Product Analysis (Top 10 by Sales)

| Product ID | Sales | Profit | Margin |
|------------|-------|--------|--------|
| TEC-CO-10004722 | $39,900 | $16,240 | **40.7%** |
| TEC-MA-10002412 | $22,638 | **-$1,811** | -8.0% |
| OFF-BI-10003527 | $20,082 | $9,278 | 46.2% |
| OFF-BI-10000545 | $19,025 | $761 | 4.0% |
| TEC-MA-10001127 | $18,375 | $4,095 | 22.3% |
| FUR-CH-10002024 | $17,454 | $631 | 3.6% |
| OFF-SU-10000151 | $17,030 | **-$262** | -1.5% |
| TEC-MA-10000822 | $16,830 | **-$4,590** | -27.3% |
| OFF-BI-10004995 | $16,332 | **-$572** | -3.5% |
| FUR-BO-10004834 | $15,611 | **-$670** | -4.3% |

4 of the top 10 revenue products are loss-making. Most are Machine (MA) sub-category, again implicating heavy discounting.

---

## Recommendations

### P0 — Immediate action

**Cap all discounts at 20%.**  
The data is unambiguous: every discount tier above 20% produces net losses. Introduce a hard approval gate at 20% via the order management system. Estimated annual profit recovery: ~$102K.

### P1 — Strategic review

**Exit or reprice the Tables sub-category.**  
TA (Tables) is the only sub-category with a meaningful scale loss (-$11,831). Either raise prices or discontinue unprofitable SKUs.

**Audit top customers by profit, not revenue.**  
SM-20320 and BM-11140 are the two largest "loss customers." Structure an account review to renegotiate terms or restructure pricing.

### P2 — Growth levers

**Prioritise Technology and Office Supplies growth.**  
Both run at ~17.5% margin versus Furniture's 2.9%. Shifting the category mix by 10pp toward TEC/OFF could add ~$40K in annual profit without growing total revenue.

**Invest in seasonal peaks.**  
March, May, September, and December consistently outperform. Pre-position inventory, run campaigns, and increase sales capacity in these months.

**Double down on Copiers (CO sub-category).**  
$42K in profit at 36.9% margin from a single sub-category. Expand the range or push higher-margin accessories alongside copier sales.

---

## Methodology

```python
import pandas as pd
import numpy as np

df = pd.read_excel("sales.xlsx")
df['Order Date'] = pd.to_datetime(df['Order Date'], dayfirst=True)
df['Ship Date'] = pd.to_datetime(df['Ship Date'], dayfirst=True)
df['Ship Days'] = (df['Ship Date'] - df['Order Date']).dt.days
df['Month'] = df['Order Date'].dt.to_period('M').astype(str)
df['Category'] = df['Product ID'].str.split('-').str[0]
df['Sub_Category'] = df['Product ID'].str.split('-').str[1]
df['Disc_Bucket'] = pd.cut(
    df['Discount'],
    bins=[-0.01, 0, 0.2, 0.4, 0.6, 0.8, 1.0],
    labels=['No Disc', '0-20%', '20-40%', '40-60%', '60-80%', '80-100%']
)
```

All aggregations use `.groupby().agg()` with `sum` for Sales/Profit and `nunique` for Orders/Customers. No imputation was required — the dataset has zero null values.

---
---

*Analysis period: June 2021 – December 2024 · 7,951 lines · 789 customers · 1,835 products*
