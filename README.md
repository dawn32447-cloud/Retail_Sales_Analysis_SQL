Project Overview

Project Title: Retail Sales Analysis

Database Name: sql_project_p2

Table Name: retail_sales

This project is a comprehensive analysis of retail sales data using SQL. It demonstrates the fundamental to advanced techniques a data analyst uses to set up a database, clean messy raw data, perform exploratory data analysis (EDA), and write analytical SQL queries to answer high-impact business questions.

The dataset includes transaction records, timestamps, customer demographics (gender and age), product categories, quantities sold, price per unit, cost of goods sold (COGS), and total revenue.


Project Objectives

Database Setup & Schema Creation: Construct the foundation database and define structured tables to hold the dataset.

Data Cleaning: Inspect the dataset to locate and eliminate entries with missing, null, or corrupted data points.

Exploratory Data Analysis (EDA): Perform high-level statistical sweeps to evaluate unique consumer counts, product categories, and baseline transaction scales.

Key Business Insights: Formulate and execute targeted SQL queries to solve ten practical retail business problems.


Repository Structure

SQL_Retail_Sales_Analysis.sql: The complete raw SQL script containing all database setup, table schemas, cleaning steps, and analysis queries.

Retail_Sales_Dataset.csv: The raw dataset used for this project.

README.md: The comprehensive project documentation you are reading right now.

Phase 1: Database Setup & Schema Design

First, we create a clean environment by establishing the schema and a primary table with constraints to prevent duplicate entries and maintain strict data integrity.

-- 1. Create the database (Run this inside your database tool first)
CREATE DATABASE sql_project_p2;

-- 2. Create the target table structure
DROP TABLE IF EXISTS retail_sales;
CREATE TABLE retail_sales (
    transactions_id INT PRIMARY KEY,
    sale_date DATE,	
    sale_time TIME,	
    customer_id INT,
    gender VARCHAR(15),
    age INT,
    category VARCHAR(15),
    quantity INT,
    price_per_unit FLOAT,	
    cogs FLOAT,
    total_sale FLOAT
);


Phase 2: Data Cleaning

Real-world datasets are rarely perfect. Before running any analytical reports, we search for records that contain missing values (NULL) in critical columns and remove them to ensure our analytical findings remain highly accurate.

-- Count total records before cleaning
SELECT COUNT(*) FROM retail_sales;

-- Query to locate records containing NULL values
SELECT * FROM retail_sales
WHERE 
    transactions_id IS NULL OR sale_date IS NULL OR sale_time IS NULL OR
    customer_id IS NULL OR gender IS NULL OR age IS NULL OR 
    category IS NULL OR quantity IS NULL OR price_per_unit IS NULL OR 
    cogs IS NULL OR total_sale IS NULL;

-- Delete records with missing data (Nulls in key numeric fields)
DELETE FROM retail_sales
WHERE 
    quantity IS NULL OR 
    price_per_unit IS NULL OR 
    cogs IS NULL OR 
    total_sale IS NULL;

-- Verify final count after cleaning
SELECT COUNT(*) FROM retail_sales;


Phase 3: Exploratory Data Analysis (EDA)

Before solving deeper business issues, we must understand the shape, scope, and diversity of our data through high-level exploratory queries.

-- 1. How many unique customers do we serve?
SELECT COUNT(DISTINCT customer_id) AS unique_customers FROM retail_sales;

-- 2. What distinct categories of products do we sell?
SELECT DISTINCT category FROM retail_sales;

-- 3. How many total orders are recorded in the dataset?
SELECT COUNT(*) AS total_orders FROM retail_sales;


Phase 4: Business Analysis & Analytical Solutions

Below are the key business questions we explored, along with the corresponding SQL queries written to retrieve the exact answers.

Q1. Retrieve all columns for sales made on '2022-11-05'

SELECT *
FROM retail_sales
WHERE sale_date = '2022-11-05';


Q2. Retrieve all transactions where the category is 'Clothing' and the quantity sold is at least 4 in the month of November 2022

SELECT *
FROM retail_sales
WHERE category = 'Clothing'
  AND TO_CHAR(sale_date, 'YYYY-MM') = '2022-11'
  AND quantity >= 4;


Q3. Calculate the total sales (net_sale) and total orders for each product category

SELECT 
    category,
    SUM(total_sale) AS net_sale,
    COUNT(*) AS total_orders
FROM retail_sales
GROUP BY category;


Q4. Find the average age of customers who purchased items from the 'Beauty' category

SELECT
    ROUND(AVG(age), 2) AS avg_age
FROM retail_sales
WHERE category = 'Beauty';


Q5. Find all transactions where the total_sale is greater than 1000

SELECT * FROM retail_sales
WHERE total_sale > 1000;


Q6. Find the total number of transactions (orders) made by each gender in each category

SELECT 
    category,
    gender,
    COUNT(*) AS total_trans
FROM retail_sales
GROUP BY category, gender
ORDER BY category ASC;


Q7. Calculate the average sale for each month and find the best-selling month in each year

SELECT year, month, avg_sale
FROM (    
    SELECT 
        EXTRACT(YEAR FROM sale_date) AS year,
        EXTRACT(MONTH FROM sale_date) AS month,
        AVG(total_sale) AS avg_sale,
        RANK() OVER(
            PARTITION BY EXTRACT(YEAR FROM sale_date) 
            ORDER BY AVG(total_sale) DESC
        ) AS rank
    FROM retail_sales
    GROUP BY 1, 2
) AS t1
WHERE rank = 1;


Q8. Find the top 5 customers based on their highest total sales contribution

SELECT 
    customer_id,
    SUM(total_sale) AS total_sales
FROM retail_sales
GROUP BY customer_id
ORDER BY total_sales DESC
LIMIT 5;


Q9. Find the number of unique customers who purchased items from each category

SELECT 
    category,    
    COUNT(DISTINCT customer_id) AS unique_customers
FROM retail_sales
GROUP BY category;


Q10. Segment orders into shift classifications (Morning <=12:00, Afternoon 12:00-17:00, Evening >17:00) and count total transactions

WITH hourly_sales AS (
    SELECT *,
        CASE
            WHEN EXTRACT(HOUR FROM sale_time) < 12 THEN 'Morning'
            WHEN EXTRACT(HOUR FROM sale_time) BETWEEN 12 AND 17 THEN 'Afternoon'
            ELSE 'Evening'
        END AS shift
    FROM retail_sales
)
SELECT 
    shift,
    COUNT(*) AS total_orders    
FROM hourly_sales
GROUP BY shift;


Key Insights & Conclusions

Category Strengths: Product categories show different levels of performance. Analyzing the breakdown helps prioritize stocking and marketing efforts for the highest-performing inventory.

Shifting Customer Habits: Our transaction times peak during the afternoon and evening hours. Concentrating staff resources, discount updates, and website updates during these busy shifts will help maximize conversions.

High-Value Loyalty: Identifying our top 5 lifetime spending consumers lets us target them with rewards, keeping them engaged and lowering customer retention costs.

Age Distributions: Calculating average shopper ages per category tells us exactly who our audience is, helping us create highly targeted marketing campaigns.
