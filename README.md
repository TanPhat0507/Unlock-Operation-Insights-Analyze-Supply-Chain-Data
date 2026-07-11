# 💥 Unlock Operation Insights – Analyze Supply Chain Data!

Understanding supply chain operations and sales performance is crucial for businesses to optimize operations and drive revenue growth. This project implements a **complete Business Intelligence workflow** — from raw data to actionable insight — analyzing key metrics such as revenue, profitability, delivery times, return rates, and customer behavior on a real-world retail supply chain dataset.

The pipeline covers the full BI lifecycle: **ETL (SSIS) → Data Warehouse & OLAP Cube (SSAS, Star Schema) → Multidimensional Querying (MDX) → Interactive Dashboards (Power BI) → Ad-hoc Analysis (Excel Pivot Tables) → Predictive Data Mining (Random Forest & CNN)** for return-rate prediction.

> Course project — University of Information Technology (UIT), VNU-HCM · MSIS4263.P22.CTTT · Lecturers: **Do Phuc**, **Nguyen Thi Kim Phung**
> Author: **Ly Tan Phat**

---

## Table of Contents

- [Overview](#overview)
- [Repository Contents](#repository-contents)
- [Dataset](#dataset)
- [BI Architecture](#bi-architecture)
  - [1. ETL with SSIS](#1-etl-with-ssis)
  - [2. Data Warehouse: Star Schema](#2-data-warehouse-star-schema)
  - [3. OLAP Cube with SSAS](#3-olap-cube-with-ssas)
  - [4. MDX Analytical Queries](#4-mdx-analytical-queries)
  - [5. Power BI Dashboards](#5-power-bi-dashboards)
  - [6. Excel Pivot Table Analysis](#6-excel-pivot-table-analysis)
  - [7. Data Mining – Return Prediction](#7-data-mining--return-prediction)
- [Key Insights](#key-insights)
- [Strategic Recommendations](#strategic-recommendations)
- [Tech Stack](#tech-stack)
- [Acknowledgements](#acknowledgements)
- [References](#references)

---

## Overview

In today's fast-paced and competitive business environment, data-driven decision-making is vital for supply chain and sales management. This project explores a retail supply chain dataset covering order processing, logistics, customer segmentation, product categories, and financial performance, in order to uncover insights that support operational efficiency, customer satisfaction, and strategic growth.

The analysis is organized around three pillars:

- 📈 **Sales analysis** — revenue/profit trends over time, by region, segment, product, and ship mode.
- 👥 **Customer analysis** — RFM segmentation to identify loyal, at-risk, and lost customers.
- 📦 **Order analysis** — delivery time, return rates by product/region/ship mode, and predictive modeling of returns.

## Repository Contents

| File | Description |
|---|---|
| `21522444_21522350_Report.docx` | Full BI report: ETL design, star schema, cube building, MDX queries, and data mining methodology. |
| `21522444_21522350_Insights.pdf` | Business insights report with charts, findings, and recommendations from the Power BI dashboards. |
| `21522444_21522350_PowerBI.pbix` | Power BI project file containing the Sales, Customer, and Order analysis dashboards. |
| `21522444_21522350_MDX.mdx` | The 10 MDX analytical queries run against the SSAS OLAP cube. |
| `ssis_project.sln` | SQL Server Integration Services solution — ETL pipeline (Extract → Transform → Load into the data warehouse). |
| `ssas_project.sln` | SQL Server Analysis Services solution — star-schema cube (`Olap Warehouse`) definition. |
| `21522444_21522350_RandomForest.ipynb` | Balanced Random Forest model predicting order returns. |
| `21522444_21522350_CNN.ipynb` | 1D-CNN model predicting order returns from tabular features. |
| `SupplyChainSalesDatasets.xlsx` | Raw source dataset (29 columns of order/shipping/customer/product records). |
| `outputPredictReturn_Pivot.xlsx` | Model output (`predict_return`) merged with the dataset, used for Excel Pivot Table analysis. |

## Dataset

- **Source:** dataset from the "Unlock Operation Insights – Analyze Supply Chain Data!" TikTok contest.
- **Time range:** 01/02/2014 – 12/30/2017
- **Country:** United States
- **Categories:** Furniture, Office Supplies, Technology
- **Size:** 9,993 raw rows across 29 columns, covering order, shipping, customer, product, sales, and return information.

## BI Architecture

### 1. ETL with SSIS

Built with **SQL Server Integration Services (SSIS)** to extract, clean, standardize, and load data into a centralized data warehouse.

- **Control Flow:** an `Execute SQL Task` truncates target tables, followed by a `Data Flow Task` for the ETL logic.
- **Extract:** raw data pulled from CSV/Excel via `Flat File Source`.
- **Transform:** data is multicast into separate branches, each sorted (e.g., by `CustomerID`, ascending, with duplicates removed) before loading into the appropriate dimension/fact table.
- **Load:** data is written into `dim_customer`, `dim_product`, `dim_order`, `dim_retailSalesPeople`, `dim_region`, `dim_returned`, and the central `fact` table via `OLE DB Destination` components.

**ETL execution results** (9,993 source rows processed):

| Table | Rows |
|---|---|
| `dim_customer` | 793 |
| `dim_product` | 1,862 |
| `dim_order` | 5,008 |
| `dim_retailSalesPeople` | 4 |
| `dim_region` | 4 |
| `dim_returned` | 2 |
| `fact` | 9,993 |

### 2. Data Warehouse: Star Schema

A **Star Schema** was chosen over a Snowflake Schema for its simplicity, faster query performance, and ease of building OLAP cubes (denormalized dimensions favor fast aggregation and drill-down).

- **Fact table:** `fact` (transactional sales/order records)
- **Dimension tables:** `Dim_Customer`, `Dim_Product`, `Dim_Order`, `Dim_Region`, `Dim_RetailSalesPeople`, `Dim_Returned`

### 3. OLAP Cube with SSAS

Built and deployed with **SQL Server Analysis Services (SSAS)**, named **`Olap Warehouse`**:

- A **Data Source** connects to the SQL Server data warehouse; a **Data Source View (DSV)** selects the relevant fact/dimension tables.
- **Measure Groups** include Sales, Cost, Quantity, and Days, with SUM/COUNT aggregations applied per business requirements.
- **Hierarchies** defined for drill-down analysis:
  - `Dim_Product`: Category → SubCategory
  - `Dim_Customer`: Country → State → City
- The cube was deployed and processed, and validated via the SSAS **Browser** (e.g., filtering `Dim Product = Furniture` to aggregate `Cost` across regions).

### 4. MDX Analytical Queries

Ten business questions were answered directly against the `Olap Warehouse` cube using **MDX (Multidimensional Expressions)**, demonstrating classic OLAP operations:

| # | Operation | Question |
|---|---|---|
| 1 | Roll-up | Total sales by category |
| 2 | Drill-down | Total sales by sub-category |
| 3 | Slice | Total sales by customer & region (East) |
| 4 | Dice | Sales by customer where Category = "Office Supplies" & Sub-Category = "Art" |
| 5 | Pivot | Total cost by region × category |
| 6 | Calculated measure | % of orders fully returned |
| 7 | Calculated measure | Return rate by region |
| 8 | Top N | Top 5 sub-categories by return rate |
| 9 | Aggregate | Maximum delivery time (days) |
| 10 | Calculated measure | Return rate by customer segment |

See [`21522444_21522350_MDX.mdx`](21522444_21522350_MDX.mdx) for the full query set.

### 5. Power BI Dashboards

The `.pbix` file (built on top of the same warehouse/cube data) contains three interactive report pages, mirroring the 10 MDX questions with visual, drillable equivalents:

- **Sales Analysis** — revenue/cost/profit/profit margin trends by year, quarter, and month; revenue by region/city, segment, product category, and ship mode.
- **Customer Analysis** — RFM (Recency, Frequency, Monetary) segmentation identifying *Potential Loyalist*, *Champions*, *At Risk*, *Lost Customers*, etc.
- **Order Analysis** — delivery time and return-rate breakdowns by time, product, region, and ship mode.

### 6. Excel Pivot Table Analysis

Using the model output in `outputPredictReturn_Pivot.xlsx`, Sales/Profit/Cost were compared between predicted-return and predicted-non-return orders via Pivot Tables and bar charts, to sanity-check and interpret the data-mining results (see [Key Insights](#key-insights)).

### 7. Data Mining – Return Prediction

Two models were trained to predict whether an order will be **returned**, using 18 features (`Ship Mode`, `Customer Name`, `Segment`, `City`, `State`, `Region`, `Retail Sales People`, `Product Name`, `Sub-Category`, `Category`, `Sales`, `Quantity`, `Discount`, `Profit`, `Cost`, `Unit CP`, `Unit SP`, `Days`), split 70% train / 30% test.

**a) Balanced Random Forest** ([`21522444_21522350_RandomForest.ipynb`](21522444_21522350_RandomForest.ipynb))
- Addresses class imbalance (far more "Not Return" than "Return" orders) using **SMOTE** oversampling + `BalancedRandomForestClassifier` (100 trees).
- **Accuracy: 88.16%**
- Top features by importance: `Discount`, `Sales`, `Profit`, `Cost`, `Days`, `Unit SP`, `Unit CP`, `Sub-Category`, `Ship Mode`, `Segment`.

**b) 1D Convolutional Neural Network** ([`21522444_21522350_CNN.ipynb`](21522444_21522350_CNN.ipynb))
- Architecture: `Conv1D(32, k=3) → Conv1D(64, k=3) → Flatten → Dense → Dense(1, sigmoid)`, trained on Min-Max-scaled features.
- **Accuracy: 91.46%**
- Top influential features (via output-layer weights): `State`, `Days`, `Segment`, `Category`, `Unit CP`, `Sales`, `Unit SP`, `Profit`, `Sub-Category`, `Cost`.

## Key Insights

**Sales**
- Revenue grew steadily from 2014–2017, with growth slowing after 2015 (29.47% → 20.36% YoY); profit margin dipped from 13.43% to 12.74% between 2014–2015.
- Revenue peaks in **Q4 / November** and troughs in **Q2 / February** — despite February having the *highest* profit margin (17.90%).
- The **West** region generates the most revenue overall (725.46K), but **New York City** (East) is the single highest-revenue city (256.37K).
- **Consumer** segment drives ~50% of revenue; **Home Office** is smallest and dipped in 2015.
- **Technology** contributes disproportionately to profit (85.5% of category profit vs. 14.5% for Office Supplies); **Furniture** has costs nearly equal to revenue, yielding thin margins.
- **Standard Class** shipping accounts for 59.12% of revenue — it's the default, lowest-cost option.

**Customers (RFM)**
- **Potential Loyalist** is the largest segment (17%, 138 customers) — recent buyers with high spend, concentrated in Furniture & Office Supplies.
- **Promising** is the smallest segment (2%, 17 customers).
- **Lost Customers** buy across all categories but mostly Furniture (48.48%) — and generated *negative* profit margin in 2014 and 2017, suggesting quality or satisfaction issues with returned Furniture purchases.

**Orders & Returns**
- Overall return rate is **5.91%** (296 of 5,009 orders); average delivery time is **34.57 days**.
- **Technology** products dominate the 100%-return-rate list (e.g., "Okidata B401 Printer", 61-day delivery time) — consistent with higher defect/dissatisfaction risk for electronics.
- The **West** region has the shortest delivery time but the *highest* return rate (11.73%).
- **Standard Class** shipping accounts for the majority (56.76%) of returned orders.
- Data mining confirms: returned orders tend to have slightly higher Sales (+$6.81) but lower Profit (−$3.42) and higher Cost (+$7.16) than non-returned orders — suggesting high-cost, low-margin items are more return-prone. `Discount`, `Sales`, `Profit`, `Cost`, and `Days` are consistently the strongest predictors of returns across both models.

## Strategic Recommendations

- 🎯 Boost **year-end promotions** (Black Friday, Cyber Monday, Christmas, New Year) to capitalize on the Q4 revenue peak; use mid-year lulls to clear inventory.
- 🌎 Focus marketing, sales, and staffing on high-revenue hubs in the **West** and key **East** cities (New York, Philadelphia); reconsider business scope in the lowest-revenue cities.
- 🛋️ Improve **Furniture** quality control and reduce production costs — thin margins and its link to Lost Customers make it a risk area.
- 💻 Expand the **Technology** portfolio (e.g., products similar to the top-selling "Canon imageCLASS 2200 Advanced Copier") while tightening QA/warranty policies to curb its high return rate.
- 🚚 Reduce shipping times, especially for **Standard Class** and in the **West** region, to lower returns and improve customer satisfaction.
- 🔁 Use the return-prediction models to proactively flag high-risk orders (high cost, high discount, long delivery time) for quality checks or personalized customer support before shipment.

## Tech Stack

| Layer | Technology |
|---|---|
| ETL | SQL Server Integration Services (SSIS) |
| Data Warehouse / OLAP Cube | SQL Server Analysis Services (SSAS), Star Schema |
| Multidimensional Querying | MDX |
| Dashboards | Microsoft Power BI |
| Ad-hoc Analysis | Excel Pivot Tables |
| Data Mining | Python, scikit-learn, imbalanced-learn (SMOTE, Balanced Random Forest), TensorFlow/Keras (CNN) |

## Acknowledgements

This project was completed as a Business Intelligence course assignment (MSIS4263.P22.CTTT) at the **University of Information Technology (UIT), VNU-HCM**. Special thanks to **Mr. Hieu Nguyen Trung (Maz)**, **Mr. To Trong Kien**, and **Ms. Phuong Vu** for their invaluable feedback, which greatly contributed to the completion of this final product.

🔗 [LinkedIn post about this project](https://www.linkedin.com/feed/update/urn:li:activity:7315234446058692608/)

## References

- Business Analyst – Hoang Long (YouTube): [link](https://www.youtube.com/watch?v=EcsaGL3AiTg&list=PLbM4yZFMwzMAdo9kSNSf-32RL5gM8vl5c&index=17)
- RFM Analysis – edrone: [link](https://help.edrone.me/en/articles/2837109-rfm-analysis)
- RFM Analysis – RetainUp: [link](https://retainup.co/rfm-analysis/)
- Basic DAX Functions – Power BI Training: [link](https://powerbitraining.com.au/basic-dax-functions-min-max-and-average/)
- Power BI Drill Through – Coupler.io: [link](https://blog.coupler.io/power-bi-drill-through/)
- Power BI COUNTIF – Coupler.io: [link](https://blog.coupler.io/power-bi-countif/)
- Waterfall Chart Guide – Gitiho: [link](https://gitiho.com/blog/huong-dan-cach-dung-va-ve-bieu-do-thac-nuoc-waterfall-chart.html)
- Funnel Chart Examples – Piktochart: [link](https://piktochart.com/tips/funnel-chart-examples)

---

### Preview

| | | |
|---|---|---|
| ![Dashboard 1](https://github.com/user-attachments/assets/2ccdd761-7ea9-4f01-ae16-f5ce6a19a31b) | ![Dashboard 2](https://github.com/user-attachments/assets/e04b0f9d-9664-41a9-8a1a-ad53fba71c5c) | ![Dashboard 3](https://github.com/user-attachments/assets/606dc428-ad56-42a7-9219-69ee464b0c11) |
