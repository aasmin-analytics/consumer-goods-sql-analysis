# consumer-goods-sql-analysis
SQL-based ad hoc analysis of AtliQ Hardware's consumer goods business, covering product portfolio, manufacturing costs, customer discounts, sales trends, and channel performance to generate actionable business insights.

# Project Overview

* This project focuses on solving ad hoc business requests for AtliQ Hardware, a global consumer electronics company. The objective is to analyze business data using SQL and present meaningful insights that can support data-driven decision-making.

The analysis covers product portfolio changes, manufacturing costs, customer discounts, sales performance, and channel contribution.

# Tools & Technologies
* SQL / MySQL: Business data querying and analysis
* PDF / PowerPoint: Presentation of findings and business insights

# Requests:
* 1.Provide the list of markets in which customer "Atliq Exclusive" operates its business in the APAC region.
  
		SELECT 
		market
		FROM gdb023.dim_customer
		where customer = "Atliq Exclusive" and region = "APAC";
  
* 2.What is the percentage of unique product increase in 2021 vs. 2020? The final output contains these fields,
	unique_products_2020 unique_products_2021 percentage_chg.

		SELECT
		    COUNT(DISTINCT CASE WHEN YEAR(date) = 2020 THEN product_code END) AS unique_products_2020,
		    COUNT(DISTINCT CASE WHEN YEAR(date) = 2021 THEN product_code END) AS unique_products_2021,
		    ROUND(
		        (
		            COUNT(DISTINCT CASE WHEN YEAR(date) = 2021 THEN product_code END)
		            - COUNT(DISTINCT CASE WHEN YEAR(date) = 2020 THEN product_code END)
		        )
		        / COUNT(DISTINCT CASE WHEN YEAR(date) = 2020 THEN product_code END) * 100,
		        2
		    ) AS percentage_chg
		FROM fact_sales_monthly;

* 3.Provide a report with all the unique product counts for each segment and 
		sort them in descending order of product counts. The final output contains
		2 fields:-  segment / product_count.
    
		Select 
			segment,
		    count(distinct(product_code)) as Product_count
		from dim_product 
		group by segment
		order by Product_count desc;

* 4.Which segment had the most increase in unique products in
 		2021 vs 2020? The final output contains these fields
		segment / product_count_2020 / product_count_2021 / difference.

		SELECT p.segment,
	    	COUNT(DISTINCT CASE WHEN YEAR(date) = 2020 THEN p.product_code END) AS unique_products_2020,
	    	COUNT(DISTINCT CASE WHEN YEAR(date) = 2021 THEN p.product_code END) AS unique_products_2021,
	    	(COUNT(DISTINCT CASE WHEN YEAR(date) = 2021 THEN p.product_code END) -
	             COUNT(DISTINCT CASE WHEN YEAR(date) = 2020 THEN p.product_code END)) as Difference
	    from fact_sales_monthly s
	    join dim_product p
			on p.product_code = s.product_code
		group by segment;

* 5.Get the products that have the highest and lowest manufacturing costs.
		 The final output should contain these fields,
   			product_code / product / manufacturing_cost.

			select 
				m.product_code, p.Product, manufacturing_cost
			from dim_product p 
			join fact_manufacturing_cost m
				on p.product_code = m.product_code
			WHERE m.manufacturing_cost = (
			    SELECT MAX(manufacturing_cost)
			    FROM fact_manufacturing_cost
			)
			OR m.manufacturing_cost = (
			    SELECT MIN(manufacturing_cost)
			    FROM fact_manufacturing_cost
			);
      

   Author
    Aasmin Khatoon
    Aspiring Data Analyst | Excel | Power BI | SQL
