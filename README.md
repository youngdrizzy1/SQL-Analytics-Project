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
- Yearly Sales
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
