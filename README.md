# E-Commerce Sales Analysis & Customer Segmentation

A data analytics project on a UK-based online retailer's transaction data (Dec 2010 – Dec 2011), covering data cleaning, exploratory analysis, and unsupervised customer segmentation using RFM + K-Means clustering.

## Business Problem

The business has ~541K raw transaction records but no clear view of who its best customers are, who's at risk of churning, and where revenue is concentrated. This project cleans the raw data and builds a customer segmentation model to answer: **which customers should the business prioritize retaining, and which are already lost?**

## Tech Stack

- **Python** (pandas, numpy) — data cleaning and feature engineering
- **scikit-learn** — K-Means clustering, StandardScaler, silhouette scoring
- **matplotlib / seaborn** — visualization
- **Jupyter Notebook** — analysis and modeling

## Project Structure

```
├── data.csv                          # raw source data
├── 01_data_cleaning.py               # cleaning pipeline
├── eda_analysis.ipynb                # exploratory analysis + RFM table
├── customer_segmentation.ipynb       # K-Means clustering model
├── 01_setup_and_import.sql           
├── 02_data_cleaning.sql
├── 03_exploratory_analysis.sql
├── requirements.txt
└── output/
    ├── audit_trail.csv               # row-count reconciliation (tracked - small, useful)
    ├── rfm_table.csv                 # tracked - small, useful for review without rerunning
    ├── rfm_with_clusters.csv         # tracked - final model output
    ├── cleaned_sales.csv             # not tracked - regenerate by running 01_data_cleaning.py
    ├── returns.csv                   # not tracked - regenerate by running 01_data_cleaning.py
    ├── non_product_transactions.csv  # not tracked - regenerate by running 01_data_cleaning.py
    ├── excluded_rows.csv             # not tracked - regenerate by running 01_data_cleaning.py
    └── charts/                       # tracked - PNG charts, referenced below
```

## Data Source

This project uses the "Online Retail" dataset - UK-based online gift retailer transactions, Dec 2010 to Dec 2011 (541,909 rows). The raw `data.csv` is not included in this repo (43MB, too large for a portfolio repo). To run this project yourself:

1. Obtain a copy of the dataset with columns: `InvoiceNo, StockCode, Description, Quantity, InvoiceDate, UnitPrice, CustomerID, Country`
2. Place it in the project root as `data.csv`
3. Follow the "How to Run" steps below

## 1. Data Cleaning

Started with **541,909** raw transaction rows. Applied the following rules, each with a documented reason rather than blanket deletion:

| Step | Rule | Rows affected |
|---|---|---|
| Deduplication | Removed exact duplicate rows | 5,268 |
| Date parsing | Converted `InvoiceDate` string to datetime | — |
| Cancellations | Flagged `InvoiceNo` starting with "C" (returns), separated into `returns.csv` rather than deleted | 9,251 |
| Non-product codes | Flagged administrative codes (POSTAGE, BANK CHARGES, AMAZONFEE, gift cards, manual entries) that aren't real products, separated into `non_product_transactions.csv` | 2,406 |
| Missing `CustomerID` | Removed — can't attribute a sale to a customer, so unusable for RFM/segmentation | majority of `excluded_rows.csv` |
| Missing `Description` | Removed | small number |
| Non-positive `Quantity` / `UnitPrice` | Removed — data entry errors or adjustments, not real sales | remainder of `excluded_rows.csv` |

**Result:** 391,150 clean, analysis-ready transaction rows. Every excluded row is preserved with a reason (in `excluded_rows.csv`, `returns.csv`, or `non_product_transactions.csv`) rather than silently dropped — the row counts reconcile exactly against the post-dedup total.

## 2. Exploratory Data Analysis

Key findings from `eda_analysis.ipynb`:

- **4,334 unique customers**, **18,402 orders**, **3,659 products**, spanning Dec 2010 – Dec 2011
- **Total revenue: £8,737,227.64**, average order value **£474.80**
- **65.3% repeat customer rate** — most buyers return at least once
- **20.85% cancellation rate** — a meaningful share of orders get returned
- Revenue trends upward through the year with a clear **November spike** (pre-holiday buying)
- **No orders on Saturdays** (likely no dispatch operations that day)
- Revenue heavily concentrated in the **UK**, with Netherlands, EIRE, and Germany a distant second/third/fourth
- Both `Frequency` and `Monetary` are strongly right-skewed — a small number of customers drive a disproportionate share of revenue

## 3. Customer Segmentation (RFM + K-Means)

Built an RFM (Recency, Frequency, Monetary) feature table per customer, then clustered with K-Means.

**Feature preparation:** `Frequency` and `Monetary` were log-transformed before scaling, since both are heavily right-skewed and would otherwise dominate the Euclidean distance calculation.

**Choosing k:** Tested k = 2 through 10 using the elbow method and silhouette score (see `charts/elbow_method.png` and `charts/silhouette_scores.png`). Selected **k = 4**, a standard, business-interpretable choice for RFM segmentation.

**Resulting segments:**

| Segment | Customers | Avg. Recency | Avg. Frequency | Avg. Monetary |
|---|---|---|---|---|
| Champions | 566 | 19.5 days | 15.8 orders | £9,719 |
| Loyal Customers | 1,443 | 45.5 days | 4.2 orders | £1,622 |
| At Risk | 938 | 260 days | 1.4 orders | £387 |
| Low Value / Lost | 1,387 | 58.6 days | 1.5 orders | £384 |

## Business Recommendations

- **Champions (566 customers, £9.7K avg spend):** Protect this segment — VIP treatment, early access to new products, loyalty perks. Losing even a handful materially impacts revenue.
- **Loyal Customers (1,443):** Growth opportunity — upsell/cross-sell campaigns to push them toward Champion-level frequency.
- **At Risk (938, ~260 days since last purchase):** Highest-priority re-engagement target — win-back email campaigns, targeted discounts, before they fully churn.
- **Low Value / Lost (1,387):** Low ROI on aggressive retention spend; consider low-cost automated re-engagement only.

## How to Run

1. Place `data.csv` in the project root.
2. Run `01_data_cleaning.py` (or the SQL scripts in order) → produces `output/cleaned_sales.csv` and related tables.
3. Run `eda_analysis.ipynb` → produces `output/rfm_table.csv` and EDA charts.
4. Run `customer_segmentation.ipynb` → produces `output/rfm_with_clusters.csv` and cluster charts.

## Dataset

UK-based online retailer transaction data, Dec 2010 – Dec 2011 (https://www.kaggle.com/datasets/carrie1/ecommerce-data).
