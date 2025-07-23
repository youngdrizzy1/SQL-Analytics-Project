# Advanced SQL Analytics Project

## Introduction
This project leverages advanced SQL techniques to analyze sales data from three tables: dim_customers, dim_products, and fact_sales. 
The goal is to answer business questions through trends, performance, and customer insights, using CTEs, window functions, and subqueries.

## Data Overview
- dim_customers: Customer details (e.g., customer_key, first_name, birthdate).
- dim_products: Product info (e.g., product_key, product_name, cost, category).
- fact_sales: Transaction records (e.g., order_date, sales_amount, customer_key, product_key).

Sample data covers March 13–18, 2013, with 498 orders, 469 customers, and 295 products.

## Analysis Tasks
1. Changes Over Time
- Yearly Sales query:
    <pre> ```SELECT
        YEAR(order_date) AS order_year,
        SUM(sales_amount) AS total_sales,
        COUNT(DISTINCT customer_key) AS total_customers,
        SUM(quantity) AS total_quantity
    FROM fact_sales
    GROUP BY YEAR(order_date)
    ORDER BY total_sales DESC;``` </pre>
  
*Result:* 2013 (up to March 18): $372,318 sales, 469 customers, 498 units.

*Insight:* Early 2013 shows robust sales; full-year data would reveal trends.

- Monthly Sales query:
    <pre> ```SELECT
        MONTH(order_date) AS order_month,
        SUM(sales_amount) AS total_sales,
        COUNT(DISTINCT customer_key) AS total_customers,
        SUM(quantity) AS total_quantity
    FROM fact_sales
    GROUP BY MONTH(order_date)
    ORDER BY total_sales DESC;``` </pre>

*Result:* March: $372,318 sales, 469 customers, 498 units.

*Insight:* March is active; broader data could confirm seasonality.

2. Cumulative Analysis
- Running Total by Year query:
  <pre> ```SELECT
    order_year,
    total_sales,
    SUM(total_sales) OVER (ORDER BY order_year) AS running_total_sales
FROM (
    SELECT
        YEAR(order_date) AS order_year,
        SUM(sales_amount) AS total_sales
    FROM fact_sales
    GROUP BY YEAR(order_date)
) AS t;``` </pre>

*Result:* 2013 running total: $372,318.

*Insight:* Steady growth in early 2013; multi-year data would show long-term patterns.

3. Performance Analysis
- Product Performance query:
  <pre> ```WITH yearly_product_sales AS (
    SELECT
        YEAR(f.order_date) AS order_year,
        p.product_name,
        SUM(f.sales_amount) AS current_sales
    FROM fact_sales AS f
    LEFT JOIN dim_products AS p ON f.product_key = p.product_key
    GROUP BY order_year, p.product_name
)``` </pre>
<pre> ```SELECT
    order_year,
    product_name,
    current_sales,
    AVG(current_sales) OVER (PARTITION BY product_name) AS avg_sales,
    current_sales - AVG(current_sales) OVER (PARTITION BY product_name) AS diff_avg,
    CASE
        WHEN current_sales > AVG(current_sales) OVER (PARTITION BY product_name) THEN 'Above Avg'
        WHEN current_sales < AVG(current_sales) OVER (PARTITION BY product_name) THEN 'Below Avg'
        ELSE 'Avg'
    END AS avg_change,
    LAG(current_sales) OVER (PARTITION BY product_name ORDER BY order_year) AS py_sales,
    current_sales - LAG(current_sales) OVER (PARTITION BY product_name ORDER BY order_year) AS diff_py,
    CASE
        WHEN current_sales > LAG(current_sales) OVER (PARTITION BY product_name ORDER BY order_year) THEN 'Increase'
        WHEN current_sales < LAG(current_sales) OVER (PARTITION BY product_name ORDER BY order_year) THEN 'Decrease'
        ELSE 'No change'
    END AS py_change
FROM yearly_product_sales
ORDER BY product_name, order_year;``` </pre>
