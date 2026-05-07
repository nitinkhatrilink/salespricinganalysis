# Sales Pricing Analysis Dashboard

An end-to-end data analytics project built using SQL and Power BI to analyze sales performance, customer behavior, product profitability, and regional trends across multiple years of transactional data.

---

# Dashboard Preview

![Dashboard Preview](https://github.com/nitinkhatrilink/bikepurchasebehavious/blob/main/Dashboard.png)

---

# Project Overview

This project focuses on transforming raw sales data using SQL and visualizing business insights through an interactive Power BI dashboard.

The workflow includes:

- Data cleaning and transformation using SQL
- Combining multi-year sales datasets
- Handling missing values
- Creating calculated business metrics
- Building interactive Power BI visualizations
- KPI analysis across products, regions, and time

---

# Tools & Technologies

- SQL Server
- Power BI
- DAX
- Data Modeling
- Data Cleaning & Transformation

---

# SQL Data Transformation (Master Query)

The core data preparation process was handled using a single master SQL query.

## Key SQL Operations Performed

### 1. Combining Multi-Year Sales Tables

Used `UNION ALL` to combine sales records from 2023, 2024, and 2025 into a unified dataset.

```sql
WITH all_orders AS (

    SELECT * FROM orders_2023
    UNION ALL
    SELECT * FROM orders_2024
    UNION ALL
    SELECT * FROM orders_2025

)
```

---

### 2. Joining Related Tables

Used `LEFT JOIN` operations to enrich transactional data with customer and product information.

```sql
LEFT JOIN customers c
ON a.customer_id = c.customer_id

LEFT JOIN products p
ON a.product_id = p.product_id
```

---

### 3. Handling Missing Revenue Values

Implemented a `CASE` statement to dynamically calculate missing revenue values.

```sql
CASE 
    WHEN revenue IS NULL 
    THEN p.price * a.quantity
    ELSE revenue
END AS cleaned_revenue
```

---

### 4. Removing Invalid Records

Filtered out incomplete records where customer information was unavailable.

```sql
WHERE customer_id IS NOT NULL
```

---

### 5. Feature Engineering for Weekly Analysis

Created a `week_date` column using `DATEADD` and `DATEDIFF` functions to aggregate daily sales into weekly trends.

```sql
DATEADD(WEEK, DATEDIFF(WEEK, 0, order_date), 0)
```

This reduced noise in Power BI line charts and improved trend readability.

---

### 6. Profit Calculation

Calculated profit directly within SQL.

```sql
revenue - cogs AS profit
```

---

# Power BI Dashboard Features

## KPI Cards

- Revenue
- Profit
- Margin %
- Average Revenue
- Quantity Sold
- Customer Count

## Interactive Filters

- Product Selection
- KPI Selector
- Dynamic Visual Switching

## Visualizations

- Revenue contribution by category
- KPI trends over time
- Regional performance analysis
- Product detail drilldowns
- Matrix table for regional sales comparison

---

# Business Insights Generated

- Identified top-performing product categories by revenue contribution
- Analyzed regional sales fluctuations over time
- Compared profitability across products and regions
- Tracked KPI trends dynamically using slicers
- Improved readability of time-series trends using weekly aggregation

---

# Skills Demonstrated

- SQL Data Transformation
- Data Cleaning
- Feature Engineering
- Relational Data Modeling
- Power BI Dashboard Design
- DAX Measures
- Business Intelligence Reporting
- KPI Analysis

---



# Future Improvements

- Add forecasting models
- Introduce customer segmentation analysis
- Include drill-through report pages
- Automate SQL refresh pipelines
