# 🛒 Sales Analytics in MySQL

This project focuses on solving real-world business problems using pure SQL queries on Atliq's sales dataset. It includes:

- Sales, Customer, Product, Market, and Date analysis
- Use of *views* *Joins* and *CTE's ,Window Functions*
- Queries grouped by business function (Sales, Customer, Market, Time,products)

## 📁 Tables Used

| Table        | Fields                                                                 
|--------------|------------------------------------------------------------------------
| customers    | customer_code, customer_name, customer_type                     
| transactions | product_code, customer_code, market_code, order_date, sales_qty, sales_amount, currency 
| products     | product_code, product_type                                         
| markets      | market_code, market_name, zone                                   
| date         | date, year, month_name, yy_mm                                 

---

## 📊 Business Problems Solved 

# SALES ANALYTICS

## Total sales amount by product type

SELECT p.product_type,SUM(t.sales_amount) AS sales
FROM products p JOIN transactions t
ON p.product_code=t.product_code GROUP BY p.product_type ORDER BY sales;

## Top 5 products by revenue

SELECT product_code, sum(sales_amount) AS revenue
FROM transactions GROUP BY product_code ORDER BY revenue desc LIMIT 5;

## Monthly sales trend

SELECT d.month_name,SUM(t.sales_amount) AS monthly_sales
FROM transactions t JOIN date d 
ON t.order_date = d.date GROUP BY d.month_name;

## Yearly sales trend

SELECT d.year,SUM(t.sales_amount) AS yearly_sales
FROM transactions t JOIN date d 
ON t.order_date = d.date GROUP BY d.year ORDER BY d.year;

## Sales on weekends

SELECT d.date, DAYNAME(d.date) AS day, SUM(t.sales_amount) AS weekend_sales
FROM transactions t JOIN date d 
ON t.order_date=d.date WHERE DAYNAME(d.date) IN ("saturday" , "sunday")
GROUP BY d.date,DAYNAME(d.date);

## Year over year growth

SELECT d.year,SUM(t.sales_amount) AS yearly_sales,
SUM(t.sales_amount) - LAG(SUM(t.sales_amount)) OVER(ORDER BY year) AS growth
FROM transactions t JOIN date d 
ON t.order_date = d.date GROUP BY d.year;

# CUSTOMER ANALYTICS

## Total customer count by type

SELECT customer_type, COUNT(*) AS total_customers
FROM customers GROUP BY customer_type;

## Top 10 customers by total spent

SELECT c.customer_code,c.custmer_name,SUM(t.sales_amount) AS total_spent 
FROM transactions t JOIN customers c 
ON t.customer_code=c.customer_code
GROUP BY c.customer_code ORDER BY total_spent LIMIT 10;

## Average sales quantity per customer

SELECT c.customer_code,c.custmer_name,AVG(t.sales_qty) AS average_sale_qty
FROM customers c JOIN transactions t 
ON c.customer_code = t.customer_code
GROUP BY c.customer_code;

## Customers with purchases in more than 1 product types

SELECT t.customer_code
FROM transactions t JOIN products p
ON t.product_code = p.product_code
GROUP BY t.customer_code
HAVING COUNT(DISTINCT p.product_type) > 1;

## Inactive Customers

SELECT c.customer_code,c.custmer_name
FROM customers c LEFT JOIN transactions t 
ON c.customer_code=t.customer_code
LEFT JOIN date d ON t.order_date=d.date
WHERE d.year IS NULL OR d.year < 2024;

# MARKET ANALYTICS

## Toatl sales by market

SELECT m.markets_code,m.markets_name, SUM(t.sales_amount) AS sales
FROM markets m JOIN transactions t
ON m.markets_code=t.market_code 
GROUP BY m.markets_code;

## Sales by Zone wise

SELECT m.zone,SUM(t.sales_amount) AS sales
FROM markets m JOIN transactions t
ON m.markets_code=t.market_code 
GROUP BY m.zone;

## Top market by sales

SELECT m.markets_code,m.markets_name,m.zone, SUM(t.sales_amount) AS sales
FROM markets m JOIN transactions t
ON m.markets_code=t.market_code 
GROUP BY m.markets_code ORDER BY sales LIMIT 1;

## Unique products sold per market

SELECT market_code, COUNT(DISTINCT product_code) AS product_variety
FROM transactions GROUP BY market_code;

## Customer count per market

SELECT market_code, COUNT(DISTINCT customer_code) AS customer_count
FROM transactions GROUP BY market_code;

# VIEWS

## Monthly sales view

CREATE VIEW monthly_sales_view AS
SELECT d.year,d.month_name,SUM(t.sales_amount) AS total_sales
FROM transactions t JOIN date d
ON t.order_date=d.date
GROUP BY d.year,d.month_name;

SELECT m.markets_code,m.markets_name, SUM(t.sales_amount) AS sales
FROM markets m JOIN transactions t
ON m.markets_code=t.market_code 
GROUP BY m.markets_code ORDER BY sales;

## Zone wise revenue view

CREATE VIEW zone_revenue_view AS
SELECT m.zone, SUM(t.sales_amount) AS total_sales
FROM transactions t JOIN markets m 
ON t.market_code= m.markets_code
GROUP BY Zone;

## Product sales view

CREATE VIEW product_sales_view AS
SELECT product_code,SUM(sales_qty) AS total_qty, SUM(sales_amount) AS total_sales
FROM transactions GROUP BY product_code;

## Top customer view ( threshold > 1000 )

CREATE VIEW top_customers AS
SELECT customer_code, SUM(sales_amount) AS total_spent
FROM transactions GROUP BY customer_code
HAVING total_spent > 1000;

# OTHER INSIGHTS

## Top product per zone

SELECT zone, product_code, total_qty 
FROM ( SELECT m.zone,t.product_code,SUM(t.sales_qty) AS total_qty,
      RANK() OVER (PARTITION BY m.zone ORDER BY SUM(t.sales_qty) DESC) AS rnk
      FROM transactions t JOIN markets m 
      ON t.market_code = m.markets_code
      GROUP BY m.zone,t.product_code) ranked
WHERE rnk = 1;

##  Average revenue per order

SELECT AVG(sales_amount) AS average_revenue_per_order
FROM transactions;

# Schema Structure

This document outlines structures for each table used in the project:

## customers
- customer_code (PK)
- customer_name
- customer_type

## transactions
- product_code
- customer_code
- market_code
- order_date (date type)
- sales_qty
- sales_amount
- currency

## products
- product_code (PK)
- product_type

## markets
- market_code (PK)
- market_name
- zone

## date
- date (PK)
- year
- month_name
- yy_mm (format: YYYY-MM)
