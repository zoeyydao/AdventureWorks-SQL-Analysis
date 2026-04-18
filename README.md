# AdventureWorks Sales Analysis Case Study

## Project Overview

AdventureWorks is a global company that sells bikes, accessories, and components worldwide. As a junior data analyst, the objective of this project was to explore transactional and dimensional data using SQL and answer business questions that support decision-making.

The analysis was conducted using three core tables:

- `dim_customers` – customer information  
- `fact_sales` – sales transactions  
- `dim_products` – product details  

---

## Business Questions

### Part 1: Getting to Know the Customers

**1.1** Start by listing all customers from Australia.
```sql
SELECT *
FROM workspace.sqlproject.gold_dim_customers
WHERE country = "Australia";
```
Result:


**1.2** The marketing team is interested in customer demographics. Show the number of customers grouped by gender.  
```sql
SELECT gender, COUNT(*) AS customer_count
FROM  workspace.sqlproject.gold_dim_customers
GROUP BY gender;
```
Result:

**1.3** Create a column that segments customers into age groups:

- `<30` = Young  
- `30–50` = Adult  
- `>50` = Senior  

Then, count how many customers fall into each group.

```sql
SELECT customer_key, customer_id, first_name, last_name, birthdate, YEAR(birthdate) AS birth_year, 
2025 - YEAR(birthdate) AS age,
CASE 
WHEN 2025 - YEAR(birthdate)<30 THEN "Young"
WHEN 2025 - YEAR(birthdate)<50 THEN "Adult"
ELSE "Senior"
END AS age_group
FROM workspace.sqlproject.gold_dim_customers;
```
Result:

```sql
SELECT
   SUM(CASE WHEN 2025 - YEAR(birthdate)<30 THEN 1 ELSE 0 END) AS below_30,
   SUM (CASE WHEN 2025 - YEAR(birthdate) BETWEEN 30 AND 49 THEN 1 ELSE 0 END) AS 30_50,
   SUM (CASE WHEN 2025 - YEAR(birthdate) > 50 THEN 1 ELSE 0 END) AS above_50
FROM workspace.sqlproject.gold_dim_customers;
```

Result:

---

### Part 2: Exploring Sales Over Time

**2.1** List all sales orders that were made in March 2013.  
```sql
SELECT *
FROM workspace.sqlproject.gold_fact_sales
WHERE YEAR(order_date) = 2013 AND MONTH(order_date) = 3;
```
Result:

**2.2** Calculate the total revenue and total quantity for each month in 2013.  
```sql
SELECT MONTH(order_date) AS month, SUM(quantity*price) AS total_revenue, SUM(quantity) AS total_quantity
FROM workspace.sqlproject.gold_fact_sales
WHERE YEAR(order_date)=2013
GROUP BY MONTH(order_date)
ORDER BY MONTH(order_date) ASC ;
```
Result:

**2.3** Find out which month in 2013 had the highest total sales amount.
```sql
SELECT MONTH(order_date) AS month, SUM(sales_amount)
FROM workspace.sqlproject.gold_fact_sales
WHERE YEAR(order_date)=2013
GROUP BY 1
ORDER BY 2 DESC
LIMIT 1;
```
Result:

---

### Part 3: Products and Categories


**3.1** Show all products that belong to the Bikes category.  
```sql
SELECT *
FROM workspace.sqlproject.gold_dim_products
WHERE category = "Bikes";
```
Result:

**3.2** Join the sales table with products and calculate the total sales amount per product name.  
```sql
SELECT p.product_name, SUM(s.sales_amount)
FROM workspace.sqlproject.gold_dim_products p
LEFT JOIN workspace.sqlproject.gold_fact_sales s
  ON p.product_key = s.product_key
GROUP BY p.product_name
ORDER BY 2;
```
Result:

**3.3** For each product category, calculate its total revenue.  
```sql
SELECT p.category, SUM(s.sales_amount)
FROM workspace.sqlproject.gold_fact_sales s
LEFT JOIN workspace.sqlproject.gold_dim_products p
  ON p.product_key = s.product_key
GROUP BY p.category
ORDER BY 2 DESC;
```
Result:

**3.4** Which five products generated the highest revenue?
```sql
SELECT p.product_name, SUM(s.sales_amount)
FROM workspace.sqlproject.gold_dim_products p
LEFT JOIN workspace.sqlproject.gold_fact_sales s
  ON p.product_key = s.product_key
GROUP BY p.product_name
ORDER BY 2 DESC
LIMIT 5;
```
Result:

---

### Part 4: Customer & Product Performance


**4.1** Join the sales and customer tables to find the top 5 customers by spending.  
```sql
SELECT c.customer_number, c.first_name, c.last_name, SUM(sales_amount) AS spending
FROM workspace.sqlproject.gold_dim_customers c
LEFT JOIN workspace.sqlproject.gold_fact_sales s
  ON c.customer_key = s.customer_key
GROUP BY c.customer_number, c.first_name, c.last_name
ORDER BY 4 DESC
LIMIT 5;
```
Result:

**4.2** For each gender, calculate the total sales revenue.  
```sql
SELECT c.gender, SUM(s.quantity*s.price) as total_revenue
FROM workspace.sqlproject.gold_dim_customers c
LEFT JOIN workspace.sqlproject.gold_fact_sales s
  ON c.customer_key = s.customer_key
GROUP BY c.gender;
```
Result:

**4.3** Segment orders into two types:

- `HighValueOrder` if sales_amount > 1000  
- `RegularOrder` otherwise  

Then, count how many orders fall into each type.
```sql
SELECT
  SUM (CASE WHEN sales_amount > 1000 THEN 1 ELSE 0 END) AS above_1000,
  SUM (CASE WHEN sales_amount <= 1000 THEN 1 ELSE 0 END) AS below_1000
FROM workspace.sqlproject.gold_fact_sales;
```
Result:

---

### Part 5: Contribution & Insights


**5.1** Calculate the contribution (%) of each product category to the company's overall revenue.  
```sql
SELECT p.category, SUM(s.sales_amount) AS category_revenue,
ROUND (SUM(s.sales_amount)*100 / (SELECT SUM(s.sales_amount) FROM workspace.sqlproject.gold_fact_sales s),2) AS contribution
FROM workspace.sqlproject.gold_fact_sales s
LEFT JOIN workspace.sqlproject.gold_dim_products p
  ON p.product_key = s.product_key
GROUP BY 1
```
Result:

**5.2** Use a subquery to find customers whose total spending is above the average customer spending.  
```sql
SELECT customer_number, first_name, last_name, SUM(sales_amount) AS Total_spending
FROM workspace.sqlproject.gold_fact_sales s
LEFT JOIN workspace.sqlproject.gold_dim_customers c
ON s.customer_key = c.customer_key
GROUP BY 1,2,3
HAVING SUM(sales_amount) > (
  SELECT AVG(Total_spending)
  FROM (
    SELECT customer_number, first_name, last_name, SUM(sales_amount) AS Total_spending
    FROM workspace.sqlproject.gold_fact_sales s
    LEFT JOIN workspace.sqlproject.gold_dim_customers c
    ON s.customer_key = c.customer_key
    GROUP BY 1,2,3
  )
)
```
Result:

**5.3** Use a subquery to find products whose revenue is above the average product revenue.
```sql
SELECT p.product_name, SUM(sales_amount) AS Total_revenue
FROM workspace.sqlproject.gold_fact_sales s
LEFT JOIN workspace.sqlproject.gold_dim_products p
ON p.product_key = s.product_key
GROUP BY 1
HAVING SUM(sales_amount) > (
  SELECT AVG(Total_revenue)
  FROM (
    SELECT p.product_name, SUM(sales_amount) AS Total_revenue
    FROM workspace.sqlproject.gold_fact_sales s
    LEFT JOIN workspace.sqlproject.gold_dim_products p
    ON p.product_key = s.product_key
    GROUP BY 1
  )
)
```
Result:

---

### Part 6: Extra Tasks


**6.1** The manager wants a list that combines all customer first names and product names into one single list. 
```sql
SELECT c.first_name AS name
FROM workspace.sqlproject.gold_dim_customers c
UNION
SELECT p.product_name AS name
FROM workspace.sqlproject.gold_dim_products p
```
Result:

**6.2** List all customer IDs who either made a purchase in 2013 OR whose country is Australia.
```sql
SELECT c.customer_number
FROM workspace.sqlproject.gold_fact_sales s
LEFT JOIN workspace.sqlproject.gold_dim_customers c
ON s.customer_key = c.customer_key
WHERE YEAR(order_date) = 2013

UNION

SELECT c.customer_number
FROM workspace.sqlproject.gold_dim_customers c
WHERE country = 'Australia';
```
Result:
