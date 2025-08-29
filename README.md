# 🍕 Pizza Sales Analysis using SQL

This project is a **SQL-based case study** designed to analyze pizza sales data and extract meaningful business insights.  
The dataset contains order-level details from a pizza store, and all queries have been written independently as part of my learning journey.

---

## 📌 Objectives

- Create and structure a relational database to store pizza sales data.  
- Populate tables using MySQL Workbench Import Wizard.  
- Perform exploratory and business-focused analysis using SQL.  
- Generate insights related to pizza categories, revenue trends, and customer ordering behavior.  

---

## 🧾 Dataset Overview

The database contains the following tables:  
- **orders** – order IDs, dates, and times  
- **order_details** – details of pizzas in each order  
- **pizzas** – pizza size and price data  
- **pizza_types** – pizza names and categories  

> **Note:**  
> - `orders` and `order_details` were created manually (large datasets imported using MySQL Workbench Import Wizard).  
> - `pizzas` and `pizza_types` were imported directly from the provided CSV files.  

---

## 📄 Complete SQL Script

```sql
-- ============================================
-- Pizza Sales Project: Full SQL Script
-- ============================================

-- 1. Create database
CREATE DATABASE pizzahut;
USE pizzahut;

-- 2. Create 'orders' table
CREATE TABLE orders (
    order_id INT NOT NULL,
    order_date DATE NOT NULL,
    order_time TIME NOT NULL,
    PRIMARY KEY (order_id)
);

-- 3. Create 'order_details' table
CREATE TABLE order_details (
    order_details_id INT NOT NULL,
    order_id INT NOT NULL,
    pizza_id TEXT NOT NULL,
    quantity INT NOT NULL,
    PRIMARY KEY (order_details_id)
);

-- ============================================
-- Data Import Notes:
-- Use MySQL Workbench Import Wizard to load 
-- 'orders' and 'order_details' data (large CSV).
-- Other tables ('pizzas', 'pizza_types') are 
-- imported directly from files.
-- ============================================

-- ============================================
-- Pizza Sales Analysis Queries
-- ============================================

-- Q.1) Retrieve the total number of orders placed.
SELECT 
    COUNT(order_id) AS total_order
FROM
    orders;

-- 2. Total revenue generated
SELECT ROUND(SUM(order_details.quantity * pizzas.price), 2) AS total_sales
FROM order_details 
JOIN pizzas ON pizzas.pizza_id = order_details.pizza_id;

-- 3. Highest-priced pizza
SELECT pizza_types.name, pizzas.price
FROM pizza_types 
JOIN pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
ORDER BY pizzas.price DESC LIMIT 1;

-- 4. Most common pizza size ordered
SELECT pizzas.size, COUNT(order_details.order_details_id) AS order_count
FROM pizzas 
JOIN order_details ON pizzas.pizza_id = order_details.pizza_id
GROUP BY pizzas.size 
ORDER BY order_count DESC;

-- 5. Top 5 most ordered pizza types
SELECT pizza_types.name, SUM(order_details.quantity) AS quantity
FROM pizza_types 
JOIN pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
JOIN order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.name 
ORDER BY quantity DESC LIMIT 5;

-- 6. Total quantity ordered by pizza category
SELECT pizza_types.category, SUM(order_details.quantity) AS quantity
FROM pizza_types 
JOIN pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
JOIN order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.category 
ORDER BY quantity DESC;

-- 7. Distribution of orders by hour of the day
SELECT HOUR(order_time) AS hour, COUNT(order_id) AS order_count
FROM orders 
GROUP BY HOUR(order_time);

-- 8. Category-wise pizza count
SELECT category, COUNT(name) 
FROM pizza_types 
GROUP BY category;

-- 9. Average number of pizzas ordered per day
SELECT ROUND(AVG(quantity), 0) AS avg_pizza_ordered_per_day
FROM (
    SELECT orders.order_date, SUM(order_details.quantity) AS quantity
    FROM orders 
    JOIN order_details ON orders.order_id = order_details.order_id
    GROUP BY orders.order_date
) AS order_quantity;

-- 10. Top 3 most ordered pizzas by revenue
SELECT pizza_types.name, SUM(order_details.quantity * pizzas.price) AS revenue
FROM pizza_types 
JOIN pizzas ON pizzas.pizza_type_id = pizza_types.pizza_type_id
JOIN order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.name 
ORDER BY revenue DESC LIMIT 3;

-- 11. Revenue contribution percentage by category
SELECT pizza_types.category,
ROUND(
    SUM(order_details.quantity * pizzas.price) / 
    (SELECT ROUND(SUM(order_details.quantity * pizzas.price),2)
     FROM order_details 
     JOIN pizzas ON pizzas.pizza_id = order_details.pizza_id) * 100, 
2) AS revenue
FROM pizza_types 
JOIN pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
JOIN order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.category 
ORDER BY revenue DESC;

-- 12. Cumulative revenue over time
SELECT order_date, SUM(revenue) OVER (ORDER BY order_date) AS cum_revenue
FROM (
    SELECT orders.order_date, SUM(order_details.quantity * pizzas.price) AS revenue
    FROM order_details 
    JOIN pizzas ON order_details.pizza_id = pizzas.pizza_id
    JOIN orders ON orders.order_id = order_details.order_id
    GROUP BY orders.order_date
) AS sales;

-- 13. Top 3 pizzas by revenue per category
SELECT name, revenue 
FROM (
    SELECT category, name, revenue,
           RANK() OVER(PARTITION BY category ORDER BY revenue DESC) AS rn
    FROM (
        SELECT pizza_types.category, pizza_types.name,
               SUM(order_details.quantity * pizzas.price) AS revenue
        FROM pizza_types 
        JOIN pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN order_details ON order_details.pizza_id = pizzas.pizza_id
        GROUP BY pizza_types.category, pizza_types.name
    ) AS a
) AS b
WHERE rn <= 3;
