# Superstore Sales & Shipping Performance Dashboard

An end-to-end Business Intelligence project: raw sales data is modeled into a star schema, transformed via a Python ETL pipeline, and visualized in an interactive Power BI dashboard delivering actionable business insights.

## Project Overview

This project takes a flat retail sales dataset and builds a complete BI pipeline around it:

1. **Data exploration & profiling** — understanding structure, quality, and content of the raw data
2. **Dimensional modeling** — designing a star schema (one fact table, five dimension tables)
3. **ETL** — a Python/pandas pipeline that transforms the flat file into the star schema
4. **Dashboard** — an interactive Power BI report with KPIs, trend analysis, and drill-down visuals
5. **Business insights** — a written analysis translating the dashboard into recommendations

## Data Source

[Sample Superstore dataset](https://www.kaggle.com/) — 9,994 order line items spanning 2014–2017, covering Sales, Profit, Customers, Products, Regions, and Shipping.

## Tools Used

- **Python (pandas)** — data profiling and ETL
- **VS Code + Jupyter** — development environment
- **Power BI Desktop** — data modeling and dashboard visualization
- **DAX** — custom measures (including a role-playing dimension handled via `USERELATIONSHIP`)

## Data Model

A star schema with one fact table and five dimension tables:

- `Fact_Sales` — one row per order line item, containing Sales, Quantity, Discount, Profit, and a derived Shipping Duration measure
- `Dim_Customer`, `Dim_Product`, `Dim_Location`, `Dim_Shipping` — standard dimensions
- `Dim_Date` — a **role-playing dimension**, referenced twice from the fact table (Order Date and Ship Date) via two relationships, only one of which is active at a time in Power BI; the inactive relationship is activated on demand using a DAX measure with `USERELATIONSHIP`

Full design rationale: [`docs/data-model.md`](docs/data-model.md)

## ETL Process

The raw flat file was profiled (no missing values, no duplicates, no anomalies), then split into fact/dimension tables using pandas, with surrogate keys generated for dimensions lacking a natural key. One real data quality issue was identified and resolved during this stage: a small number of Product IDs had inconsistent Product Name/Category values across rows, requiring deduplication to enforce a unique dimension key.

Full process details: [`docs/etl-process.md`](docs/etl-process.md)

## Dashboard

The Power BI dashboard includes:
- KPI cards: Total Sales, Total Profit, Total Orders (distinct count), Total Quantity
- Sales trend over time (revealing clear seasonality, peaking in Nov/Dec)
- Profit by Category (revealing Furniture as a comparatively weak-margin category)
- Sales by Region (West/East outperforming Central/South)
- Average Shipping Duration by Ship Mode (confirming shipping tiers meet their expected timeframes)
- A Year slicer for interactive filtering across all visuals

![Dashboard Screenshot](assets/dashboard-screenshot.png)

*(Add your screenshot and slicer GIF to an `assets/` folder and update the paths above.)*

## Key Business Insights

- Sales are strongly seasonal, with a consistent Nov/Dec peak across all four years — suggesting inventory and staffing should be planned around this cycle
- Furniture underperforms on profit relative to its sales volume, likely driven by specific sub-categories
- The South region lags noticeably behind other regions, representing either a market-size limitation or a growth opportunity
- Shipping modes reliably deliver within their expected timeframes, a strong operational finding

Full write-up: [`docs/business-insights.md`](docs/business-insights.md)

## Repository Structure

```
BI-Sales-Project/
├── explore.ipynb              # ETL and profiling notebook
├── Sample_Superstore.csv.xlsx # raw source data
├── Fact_Sales.csv
├── Dim_Date.csv
├── Dim_Customer.csv
├── Dim_Product.csv
├── Dim_Location.csv
├── Dim_Shipping.csv
├── dashboard.pbix             # Power BI report file
├── docs/
│   ├── data-model.md
│   ├── etl-process.md
│   └── business-insights.md
└── assets/
    ├── dashboard-screenshot.png
    └── slicer-demo.gif
```

## Skills Demonstrated

Dimensional modeling (star schema, role-playing dimensions), data profiling and cleaning, Python/pandas ETL scripting, Power BI data modeling and DAX, business-focused data storytelling.
