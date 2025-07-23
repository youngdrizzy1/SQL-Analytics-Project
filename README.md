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
    GROUP BY order_year, p.product_name)``` </pre>
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

    *Result:* Top products: "Mountain-200 Black- 38" ($2,295), "Touring-1000 Blue- 46" ($2,384).

    *Insight:* Premium bikes lead sales; multi-year data needed for full comparison.

4. Proportional Analysis
- Category Contribution query:
    <pre> ```WITH category_sales AS (
    SELECT
        p.category,
        SUM(f.sales_amount) AS total_sales
    FROM fact_sales AS f
    LEFT JOIN dim_products AS p ON p.product_key = f.product_key
    GROUP BY p.category)``` </pre>
    <pre> ```SELECT
        category,
        total_sales,
        SUM(total_sales) OVER () AS overall_sales,
        CONCAT(ROUND((CAST(total_sales AS FLOAT) / SUM(total_sales) OVER ()) * 100, 2), '%') AS percent_of_total
    FROM category_sales
    ORDER BY total_sales DESC;``` </pre>

    *Result:* Bikes ~92.7% ($345,000), Accessories ~7.3% ($27,000).

    *Insight:* Bikes dominate revenue, suggesting a focus on high-value products.

5. Data Segmentation
- Product Cost Ranges query:
  <pre> ```WITH product_segments AS (
    SELECT
        product_key,
        product_name,
        cost,
        CASE 
            WHEN cost < 100 THEN 'Below 100'
            WHEN cost BETWEEN 100 AND 500 THEN '100 - 500'
            WHEN cost BETWEEN 500 AND 1000 THEN '500 - 1000'
            ELSE 'Above 1000'
        END AS cost_range
    FROM dim_products)``` </pre>
    <pre> ```SELECT
        cost_range,
        COUNT(product_key) AS total_products
    FROM product_segments
    GROUP BY cost_range
    ORDER BY total_products DESC;``` </pre>

    *Result:* Below 100: 117, 100–500: 92, 500–1000: 36, Above 1000: 50.

    *Insight:* Low-cost products are numerous, but high-cost bikes drive value.

- Customer Segments query:
    <pre> ```WITH customer_spending AS (
    SELECT
        c.customer_key,
        SUM(f.sales_amount) AS total_spending,
        MIN(f.order_date) AS first_order,
        MAX(f.order_date) AS last_order,
        TIMESTAMPDIFF(MONTH, MIN(f.order_date), MAX(f.order_date)) AS lifespan
    FROM fact_sales f
    LEFT JOIN dim_customers c ON f.customer_key = c.customer_key
    GROUP BY c.customer_key)``` </pre>
    <pre> ```SELECT
    customer_segment,
    COUNT(customer_key) AS total_customers
    FROM (
    SELECT
        customer_key,
        total_spending,
        lifespan,
        CASE 
            WHEN lifespan >= 12 AND total_spending > 5000 THEN 'VIP'
            WHEN lifespan >= 12 AND total_spending <= 5000 THEN 'Regular'
            ELSE 'New'
    END AS customer_segment
    FROM customer_spending) AS segments
    GROUP BY customer_segment;``` </pre>

    *Result:* New: 469 customers (lifespan 0 months).
  
    *Insight:* All customers are new in this period; longer data would show loyalty.

6. Customer Report
    <pre> ```WITH base_query AS (
    SELECT
        f.order_number,
        f.product_key,
        f.order_date,
        f.sales_amount,
        f.quantity,
        c.customer_key,
        c.customer_number,
        CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
        TIMESTAMPDIFF(YEAR, c.birthdate, CURDATE()) AS age
    FROM fact_sales f
    LEFT JOIN dim_customers c ON f.customer_key = c.customer_key
    WHERE f.order_date IS NOT NULL),``` </pre>
    <pre> ```,customer_aggregation AS (
    SELECT
        customer_key,
        customer_number,
        customer_name,
        age,
        COUNT(DISTINCT order_number) AS total_orders,
        SUM(sales_amount) AS total_sales,
        SUM(quantity) AS total_quantity,
        COUNT(DISTINCT product_key) AS total_products,
        MAX(order_date) AS last_order_date,
        TIMESTAMPDIFF(MONTH, MIN(order_date), MAX(order_date)) AS lifespan
    FROM base_query
    GROUP BY customer_key, customer_number, customer_name, age)``` </pre>
    <pre> ```SELECT
    customer_key,
    customer_number,
    customer_name,
    age,
    CASE
        WHEN age < 20 THEN 'Under 20'
        WHEN age BETWEEN 20 AND 29 THEN '20-29'
        WHEN age BETWEEN 30 AND 39 THEN '30-39'
        WHEN age BETWEEN 40 AND 49 THEN '40-49'
        ELSE '50 and above'
    END AS age_group,
    lifespan,
    CASE 
        WHEN lifespan >= 12 AND total_sales > 5000 THEN 'VIP'
        WHEN lifespan >= 12 AND total_sales <= 5000 THEN 'Regular'
        ELSE 'New'
    END AS customer_segment,
    total_orders,
    total_sales,
    total_quantity,
    total_products,
    last_order_date,
    TIMESTAMPDIFF(MONTH, last_order_date, CURDATE()) AS recency,
    CASE 
        WHEN total_orders = 0 THEN 0
        ELSE total_sales / total_orders
    END AS avg_order_value,
    CASE 
        WHEN lifespan = 0 THEN total_sales
        ELSE total_sales / lifespan
    END AS avg_monthly_spend
    FROM customer_aggregation;``` </prev>
