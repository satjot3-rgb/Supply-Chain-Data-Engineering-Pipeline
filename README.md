# DataCo Supply Chain Data Pipeline

> A Medallion Architecture pipeline built with PySpark & Delta Lake on Google Colab

![PySpark](https://img.shields.io/badge/PySpark-3.5.4-E25A1C?style=flat&logo=apachespark&logoColor=white)
![Delta Lake](https://img.shields.io/badge/Delta_Lake-3.2.0-00ADD8?style=flat)
![Python](https://img.shields.io/badge/Python-3.x-3776AB?style=flat&logo=python&logoColor=white)
![Platform](https://img.shields.io/badge/Platform-Google_Colab-F9AB00?style=flat&logo=googlecolab&logoColor=white)
![Power BI](https://img.shields.io/badge/Visualization-Power_BI-F2C811?style=flat&logo=powerbi&logoColor=black)

---

## Overview

This project implements a full end-to-end data engineering pipeline using the **DataCo Smart Supply Chain** dataset from Kaggle. Raw transactional supply chain data is processed through three structured layers — **Bronze**, **Silver**, and **Gold** — following the **Medallion Architecture** pattern. The final Gold layer outputs are designed for direct consumption in Power BI dashboards.

---

## Dataset

The dataset is too large to host on GitHub. Please download it from Kaggle before running the notebook:

**Download:** [DataCo Smart Supply Chain Dataset on Kaggle](https://www.kaggle.com/code/kerneler/starter-dataco-smart-supply-chain-for-407715b4-c/input)

After downloading, place `DataCoSupplyChainDataset.csv` in the same directory as the notebook before running.

---

## Tech Stack

| Component | Technology | Version |
|-----------|------------|---------|
| Processing Engine | Apache Spark (PySpark) | 3.5.4 |
| Table Format | Delta Lake | 3.2.0 |
| Runtime Environment | Google Colab | — |
| Language | Python | 3.x |
| Dependency Manager | Findspark | Latest |
| Visualization | Power BI | — |

---

## Pipeline Architecture — Medallion Layers

```
Raw CSV
   │
   ▼
┌─────────────────────────────┐
│         BRONZE LAYER        │  Raw ingestion → Delta table (no transforms)
│     /mnt/bronze/dataco_raw  │
└─────────────┬───────────────┘
              │
              ▼
┌─────────────────────────────┐
│         SILVER LAYER        │  Cleaning, renaming, KPI derivation
│       /mnt/silver/orders    │
└─────────────┬───────────────┘
              │
              ▼
┌─────────────────────────────┐
│          GOLD LAYER         │  Business aggregations → CSV exports
│   monthly_sales.csv         │
│   category_sales.csv        │
│   shipping_performance.csv  │
│   top_customers.csv         │
└─────────────────────────────┘
```

---

### Bronze Layer — Raw Ingestion

The Bronze layer ingests the raw CSV dataset with no transformations, preserving the original data in its entirety as a Delta table. Column mapping is enabled to handle column names containing spaces and special characters.

- Reads CSV with schema inference and header detection
- Writes to Delta format with column mapping enabled (`delta.columnMapping.mode = name`)
- Stored at: `/mnt/bronze/dataco_raw`

---

### Silver Layer — Data Cleaning & Transformation

The Silver layer loads data from Bronze and applies quality checks, cleaning, renaming, and feature engineering to produce an analytics-ready table.

**Steps performed:**

- Duplicate row detection and removal via `dropDuplicates()`
- Null analysis across all columns using `count(when(isNull))`
- Null imputation for `Sales`, `Profit`, and `Benefit` columns with `0`
- Column renaming for consistency:

  | Original Column | Renamed To |
  |-----------------|------------|
  | Order Id | order_id |
  | Customer Id | customer_id |
  | Product Card Id | product_id |
  | Product Name | product_name |
  | Category Name | category_name |
  | Sales | sales_amount |
  | Order Profit Per Order | profit_per_order |

- Date transformation: order date parsed to timestamp; `order_year` and `order_month` extracted
- KPI derivation: `delivery_delay_days = Days for shipping (real) − Days for shipment (scheduled)`
- Stored at: `/mnt/silver/orders`

---

### Gold Layer — Business Aggregations & Exports

The Gold layer builds analytical aggregates directly consumed by Power BI reports. Four tables are generated and exported as CSV:

| File | Description | Key Columns |
|------|-------------|-------------|
| `monthly_sales.csv` | Revenue aggregated by year and month | order_year, order_month, revenue |
| `category_sales.csv` | Revenue and profit by product category | category_name, revenue, profit |
| `shipping_performance.csv` | Average delivery delay by shipping mode | Shipping Mode, avg_delay |
| `top_customers.csv` | Total spend ranked by customer | customer_id, total_sales |

An additional **window function analysis** computes month-over-month sales growth percentage per product using `lag()` with a partitioned `Window` spec over `product_id`, `order_year`, and `order_month`.

---

## Repository Structure

```
supply-chain-pipeline/
├── notebook / Supply_Chain_Pipeline.ipynb       # Main PySpark notebook
├── data / monthly_sales.csv                     # Gold: Monthly revenue output
├── data / category_sales.csv                    # Gold: Category performance output
├── data / shipping_performance.csv              # Gold: Shipping delay output
├── data / top_customers.csv                     # Gold: Top customers output
├── dashboard / temp 
└──README.md                       # Project documentation

# Note: Dataset not included — download from the Kaggle link above
```

---

## How to Run

1. Open the notebook in **Google Colab**
2. Download the dataset from the Kaggle link above and upload `DataCoSupplyChainDataset.csv` to the Colab session
3. Run the first cell (**Environment Setup**) — this installs Java 11, Spark 3.5.4, PySpark, and Delta Lake
4. If a `JavaPackage` error occurs after the first run, go to **Runtime → Restart Session**, then re-run the setup cell
5. Run all subsequent cells in order: **Bronze → Silver → Gold**
6. Four CSV files will be generated in the Colab working directory — download these for Power BI

---

## Power BI Integration

The four Gold layer CSV files serve as data sources for the Power BI dashboard. Load each file using **Get Data → Text/CSV** in Power BI Desktop.

| CSV File | Recommended Visual |
|----------|--------------------|
| `monthly_sales.csv` | Line chart — Revenue trend over time (Year/Month on X axis) |
| `category_sales.csv` | Clustered bar chart — Revenue vs Profit by category |
| `shipping_performance.csv` | Bar chart — Average delay by shipping mode |
| `top_customers.csv` | Table or treemap — Top N customers by total spend |

---

## Key Concepts Demonstrated

- **Medallion Architecture** (Bronze / Silver / Gold) for incremental data refinement
- **Delta Lake** for ACID-compliant, versioned table storage
- **PySpark DataFrame API** for large-scale distributed data processing
- **Column Mapping** in Delta to preserve original column names with spaces
- **Window Functions** with `lag()` for time-series growth calculations
- **KPI Engineering**: delivery delay, revenue, and profit margin derivation
- **Data Quality**: null detection, imputation, and deduplication patterns

---

## Dataset Credit

DataCo Smart Supply Chain for Big Data Analysis — available on [Kaggle](https://www.kaggle.com/code/kerneler/starter-dataco-smart-supply-chain-for-407715b4-c/input)

---

*Built using PySpark 3.5.4 + Delta Lake 3.2.0 on Google Colab*
