# Sales Analytics Master SQL Query

```sql
/* =========================================================
   SALES ANALYTICS MASTER QUERY
   ========================================================= */

-- =========================================================
-- 1. COMBINE MULTI-YEAR ORDER DATA
-- =========================================================

WITH all_orders AS (

    SELECT order_id,
           customer_id,
           product_id,
           order_date,
           quantity,
           revenue,
           cogs
    FROM orders_2023

    UNION ALL

    SELECT order_id,
           customer_id,
           product_id,
           order_date,
           quantity,
           revenue,
           cogs
    FROM orders_2024

    UNION ALL

    SELECT order_id,
           customer_id,
           product_id,
           order_date,
           quantity,
           revenue,
           cogs
    FROM orders_2025
),

-- =========================================================
-- 2. REMOVE INVALID / DUPLICATE RECORDS
-- =========================================================

clean_orders AS (

    SELECT DISTINCT *
    FROM all_orders
    WHERE customer_id IS NOT NULL
      AND product_id IS NOT NULL
      AND quantity > 0
),

-- =========================================================
-- 3. ENRICH DATA USING DIMENSION TABLES
-- =========================================================

enriched_orders AS (

    SELECT
        a.order_id,
        a.customer_id,
        a.product_id,
        a.order_date,
        a.quantity,
        a.revenue,
        a.cogs,

        c.region,
        c.customer_join_date,

        p.product_name,
        p.product_category,
        p.price,
        p.base_cost

    FROM clean_orders a

    LEFT JOIN customers c
        ON a.customer_id = c.customer_id

    LEFT JOIN products p
        ON a.product_id = p.product_id
),

-- =========================================================
-- 4. FEATURE ENGINEERING & DATA CLEANING
-- =========================================================

transformed_data AS (

    SELECT

        order_id,
        customer_id,
        product_id,
        order_date,

        -- WEEK START DATE
        DATEADD(WEEK, DATEDIFF(WEEK, 0, order_date), 0)
            AS week_date,

        -- MONTH & YEAR EXTRACTION
        DATENAME(MONTH, order_date)
            AS order_month,

        YEAR(order_date)
            AS order_year,

        DATEPART(QUARTER, order_date)
            AS order_quarter,

        quantity,

        -- HANDLE MISSING REVENUE
        CASE
            WHEN revenue IS NULL
                THEN price * quantity
            ELSE revenue
        END AS cleaned_revenue,

        cogs,

        -- PROFIT METRIC
        CASE
            WHEN revenue IS NULL
                THEN (price * quantity) - cogs
            ELSE revenue - cogs
        END AS profit,

        -- PROFIT MARGIN %
        ROUND(
            (
                CASE
                    WHEN revenue IS NULL
                        THEN ((price * quantity) - cogs)
                    ELSE (revenue - cogs)
                END
            ) * 100.0
            /
            NULLIF(
                CASE
                    WHEN revenue IS NULL
                        THEN (price * quantity)
                    ELSE revenue
                END,
            0),
        2) AS profit_margin_pct,

        -- CUSTOMER TENURE
        DATEDIFF(
            DAY,
            customer_join_date,
            order_date
        ) AS customer_tenure_days,

        -- CUSTOMER SEGMENTATION
        CASE
            WHEN quantity >= 10
                THEN 'Bulk Buyer'

            WHEN quantity >= 5
                THEN 'Medium Buyer'

            ELSE 'Low Buyer'
        END AS customer_segment,

        region,
        product_name,
        product_category,
        price,
        base_cost

    FROM enriched_orders
),

-- =========================================================
-- 5. ADVANCED ANALYTICS USING WINDOW FUNCTIONS
-- =========================================================

final_dataset AS (

    SELECT

        *,

        -- RUNNING REVENUE
        SUM(cleaned_revenue)
        OVER (
            PARTITION BY order_year
            ORDER BY order_date
        ) AS cumulative_revenue,

        -- PRODUCT SALES RANK
        DENSE_RANK()
        OVER (
            PARTITION BY order_year
            ORDER BY cleaned_revenue DESC
        ) AS revenue_rank,

        -- 7-DAY MOVING AVERAGE
        AVG(cleaned_revenue)
        OVER (
            ORDER BY order_date
            ROWS BETWEEN 6 PRECEDING
            AND CURRENT ROW
        ) AS moving_avg_7day_revenue

    FROM transformed_data
)

-- =========================================================
-- 6. FINAL OUTPUT
-- =========================================================

SELECT *
FROM final_dataset;
```
