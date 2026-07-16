# Data Model Design — BI Sales Project

## 1. Overview

This document describes the dimensional data model used for the BI Sales Project, based on the Sample Superstore dataset (9,994 rows, 19 columns, sourced as a flat file with no missing values or duplicates).

The raw data was denormalized into a **star schema**, a modeling convention widely used in data warehousing and business intelligence because it optimizes for fast aggregation and simple, intuitive joins for reporting tools like Power BI or Tableau — as opposed to a normalized (3NF) design, which is optimized for transactional systems rather than analytics.

The schema consists of one central fact table and five supporting dimension tables.

---

## 2. Fact Table

### Fact_Sales
**Grain:** one row per order line item (i.e. one product within one order).

| Column | Type | Description |
|---|---|---|
| Row ID | PK | Unique identifier for the line item, inherited from source data |
| Order ID | | Groups line items belonging to the same order |
| Customer ID | FK → Dim_Customer | |
| Product ID | FK → Dim_Product | |
| Order Date ID | FK → Dim_Date | Date the order was placed |
| Ship Date ID | FK → Dim_Date | Date the order shipped |
| Location ID | FK → Dim_Location | |
| Ship Mode ID | FK → Dim_Shipping | |
| Sales | Measure | Revenue for the line item |
| Quantity | Measure | Units sold |
| Discount | Measure | Discount applied (proportion) |
| Profit | Measure | Profit/loss for the line item |

**Design choice:** Measures (Sales, Quantity, Discount, Profit) stay in the fact table because they are numeric and additive — they can be summed, averaged, or aggregated across any dimension. Everything else that describes *who, what, where, or when* was extracted into dimension tables, following standard star-schema convention of separating quantitative facts from descriptive context.

---

## 3. Dimension Tables

### Dim_Customer
| Column | Type |
|---|---|
| Customer ID | PK |
| Segment | |

**Note:** The source data contains only `Customer ID`, with no separate customer name field. This is documented here rather than silently ignored, since it affects what customer-level reporting is possible (customers are identifiable only by ID, not name).

### Dim_Product
| Column | Type |
|---|---|
| Product ID | PK |
| Product Name | |
| Category | |
| Sub-Category | |

### Dim_Location
| Column | Type |
|---|---|
| Location ID | PK (surrogate key, generated) |
| Country | |
| State | |
| City | |
| Region | |

**Design choice:** Country, State, City, and Region are combined into a single dimension rather than split into separate tables (a "snowflake" approach), since geographic attributes are typically queried together and combining them keeps the schema simpler — a reasonable star-schema simplification for a dataset of this size and scope.

### Dim_Date
| Column | Type |
|---|---|
| Date ID | PK (surrogate key, generated) |
| Full Date | |
| Year | |
| Quarter | |
| Month | |
| Month Name | |
| Day | |
| Weekday | |
| Is Weekend | Boolean |

**Design choice:** Date dimensions are not derived from a single source column but generated independently (a standard BI practice) so that time-based attributes — quarter, weekday, weekend flag — are available for slicing without repeated calculation in reports.

**Role-playing dimension:** `Dim_Date` is referenced twice from the fact table — once as **Order Date** and once as **Ship Date** — via two separate foreign keys (`Order Date ID`, `Ship Date ID`). This is known as a role-playing dimension: the same physical table serves two distinct analytical roles. It avoids duplicating the date table while still allowing both dates to be analyzed independently (e.g. filtering by order month vs. filtering by ship month).

### Dim_Shipping
| Column | Type |
|---|---|
| Ship Mode ID | PK (surrogate key, generated) |
| Ship Mode | e.g. Standard Class, Second Class, First Class, Same Day |

**Design choice:** Modeled as its own dimension, rather than left as a plain text column in the fact table, to support shipping-specific analysis (e.g. average shipping duration or profit by shipping mode) and to keep the fact table normalized against repeating text values.

---

## 4. Derived Measure: Shipping Duration

Not present in the source data — calculated during ETL as:

```
Shipping Duration (days) = Ship Date − Order Date
```

This is added to Fact_Sales as an additional measure. It enables analysis such as *"which shipping mode has the longest average delay?"* — a genuinely useful business question that connects two dimensions (Date and Shipping) through the fact table, since dimension tables are never linked directly to one another in a star schema.

---

## 5. Schema Diagram (textual)

```
                    Dim_Date
                   (role-playing:
                 Order Date / Ship Date)
                        |
Dim_Customer ---- Fact_Sales ---- Dim_Product
                        |
              Dim_Location   Dim_Shipping
```

All dimension tables connect only to Fact_Sales, never to each other directly — the defining characteristic of a star schema, as opposed to a snowflake schema where dimensions can reference other dimensions.
