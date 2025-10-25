# Retail Sales Analysis SQL Project

## Project Overview

**Project Title**: Retail Sales Analysis  
**Level**: Beginner  
**Database**: `p1_retail_db`

This project is designed to demonstrate SQL skills and techniques typically used by data analysts to explore, clean, and analyze retail sales data. The project involves setting up a retail sales database, performing exploratory data analysis (EDA), and answering specific business questions through SQL queries. This project is ideal for those who are starting their journey in data analysis and want to build a solid foundation in SQL.

## Objectives

1. **Set up a retail sales database**: Create and populate a retail sales database with the provided sales data.
2. **Data Cleaning**: Identify and remove any records with missing or null values.
3. **Exploratory Data Analysis (EDA)**: Perform basic exploratory data analysis to understand the dataset.
4. **Business Analysis**: Use SQL to answer specific business questions and derive insights from the sales data.

## Project Structure

### 1. Database Setup

- **Database Creation**: The project starts by creating a database named `p1_retail_db`.
- **Table Creation**: A table named `retail_sales` is created to store the sales data. The table structure includes columns for transaction ID, sale date, sale time, customer ID, gender, age, product category, quantity sold, price per unit, cost of goods sold (COGS), and total sale amount.

```sql
CREATE DATABASE p1_retail_db;

CREATE TABLE retail_sales
(
    transactions_id INT PRIMARY KEY,
    sale_date DATE,	
    sale_time TIME,
    customer_id INT,	
    gender VARCHAR(10),
    age INT,
    category VARCHAR(35),
    quantity INT,
    price_per_unit FLOAT,	
    cogs FLOAT,
    total_sale FLOAT
);
```
### 2. Load the data
A bulk retail sales dataset has been successfully loaded into SQL Server Management Studio for analysis. The dataset contains detailed sales transaction records, including date, time, product, quantity, price, and payment details. It will be used to perform sales trend analysis, identify peak sales periods, and generate business insights.
```
BULK INSERT [dbo].[retail_sales]
FROM 'C:\Users\abina\OneDrive\Documents\SQL Server Management Studio\SQL - Retail Sales Analysis_utf .csv'
WITH (
    FORMAT = 'CSV',         -- Optional in latest versions
    FIRSTROW = 2,           -- Skip header
    FIELDTERMINATOR = ',', 
    ROWTERMINATOR = '\n',
    TABLOCK
);
```
### 2. Data Exploration & Cleaning

This stage focuses on examining the structure and quality of the Retail Sales dataset after performing a bulk load into SQL Server Management Studio (SSMS). The main objectives include identifying record counts, unique entities, and handling missing data to ensure data accuracy and consistency for further analysis.

#### Data Exploration

* **View Dataset:**

  ```sql
  SELECT * FROM retail_sales;
  ```

  Displays all records from the `retail_sales` table for an initial review of data structure and values.

* **Total Record Count:**

  ```sql
  SELECT COUNT(*) AS total_records FROM retail_sales;
  ```

  Retrieves the total number of transactions (rows) in the dataset.

* **Total Sales Count:**

  ```sql
  SELECT COUNT(*) AS total_sale FROM retail_sales;
  ```

  Returns the total number of sales transactions available.

* **Unique Customer Count:**

  ```sql
  SELECT COUNT(DISTINCT customer_id) AS unique_customers FROM retail_sales;
  ```

  Determines the number of unique customers who made purchases.

* **Unique Product Categories:**

  ```sql
  SELECT DISTINCT category FROM retail_sales;
  ```

  Lists all unique product categories available in the dataset.

#### **Data Cleaning**

To ensure data quality, null values across all key fields were identified and removed.

* **Check for Null Values:**

  ```sql
  SELECT * FROM retail_sales
  WHERE transactions_id IS NULL
     OR sale_date IS NULL
     OR sale_time IS NULL
     OR customer_id IS NULL
     OR gender IS NULL
     OR category IS NULL
     OR quantiy IS NULL
     OR price_per_unit IS NULL
     OR cogs IS NULL
     OR total_sale IS NULL;
  ```

* **Delete Records with Missing Data:**

  ```sql
  DELETE FROM retail_sales
  WHERE transactions_id IS NULL
     OR sale_date IS NULL
     OR sale_time IS NULL
     OR customer_id IS NULL
     OR gender IS NULL
     OR category IS NULL
     OR quantiy IS NULL
     OR price_per_unit IS NULL
     OR cogs IS NULL
     OR total_sale IS NULL;
  ```

  Removes any incomplete records to maintain data integrity before analysis.



-- Data Analysis & Business key problems
-- 1. Write a SQL query to retrieve all columns for sales made on '2022-11-05'
```sql
 Select * 
 from retail_sales
 where sale_date =  '2022-11-05'
```

 -- 2.Write a SQL query to retrieve all transactions where the category is 'Clothing' and the quantity sold is more than 4 in the month of Nov-2022

 ```sql
 Select *
 from retail_sales
 where category='Clothing'
 and
FORMAT(sale_date, 'yyyy-MM') = '2022-11'
 and
 quantiy>=4
 ```

-- 3.Write a SQL query to calculate the total sales (total_sale) for each category

``` sql
Select category, SUM(total_sale) as total_sale_group 
from retail_sales
group by category
```

-- 4. Write a SQL query to find the average age of customers who purchased items from the 'Beauty' category.
``` sql
Select AVG(age) as avg_age from retail_sales
where category='Beauty'
```

-- 5. Write a SQL query to find all transactions where the total_sale is greater than 1000.
``` sql
Select * from retail_sales
where total_sale>1000
```

-- 6.Write a SQL query to find the total number of transactions (transaction_id) made by each gender in each category.
``` sql
SELECT 
    category,
    gender,
    COUNT(transactions_id) AS total_transactions
FROM 
    retail_sales
GROUP BY 
    category, gender;
```

-- 7. Write a SQL query to calculate the average sale for each month. Find out best selling month in each year

``` sql
Select
	year(sale_date) as year,
	month(sale_date) as month,
	avg(total_sale) as avg_sales
from
    retail_sales
group by
	year(sale_date),
	month(sale_date)
order by
    year,month

With ranked_sales as(
	Select
		YEAR(sale_date) as sale_year,
		MONTH(sale_date) as sale_month,
		SUM(total_sale) as total_sales,
		RANK() over(partition by year(sale_date) order by sum(total_sale) desc) as rank_in_year
		from retail_sales
		group by
			YEAR(sale_date),
			month(sale_date)
		)
Select
	sale_year,
	sale_month,
	total_sales
from ranked_sales
where rank_in_year = 1;
```

-- 8.Write an sql query to find the top 5 customers based on the highest total sales.
``` sql
SELECT TOP 5
    customer_id,
    SUM(total_sale) AS total_sales
FROM retail_sales
GROUP BY customer_id
ORDER BY total_sales DESC;

```

--9.Write an sql query to find the number of unique customers who purchased items from each category.
``` sql
Select category,
count(distinct customer_id) as total_customers
from retail_sales
group by category

```

--10. Write an sql query to create each shiftand number of orders(Example Morning<=12,Afternoon between 12 & 17, Evening >17.

``` sql
SELECT 
    CASE
        WHEN DATEPART(HOUR, sale_time) < 12 THEN 'Morning'
        WHEN DATEPART(HOUR, sale_time) BETWEEN 12 AND 17 THEN 'Afternoon'
        ELSE 'Evening'
    END AS shift,
    COUNT(*) AS number_of_orders
FROM retail_sales
GROUP BY 
    CASE
        WHEN DATEPART(HOUR, sale_time) < 12 THEN 'Morning'
        WHEN DATEPART(HOUR, sale_time) BETWEEN 12 AND 17 THEN 'Afternoon'
        ELSE 'Evening'
    END;
```
-- END OF PROJECT
