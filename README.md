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

* 6. Generate a report which contains the top 5 customers who received an
		average high pre_invoice_discount_pct for the fiscal year 2021 and in the
		Indian market. The final output contains these fields,
		customer_code / customer / average_discount_percentage

			SELECT
			    c.customer_code,
			    c.customer,
			    ROUND(AVG(pre_invoice_discount_pct), 2) AS average_discount_percentage
			FROM fact_pre_invoice_deductions d
			JOIN dim_customer c
			    ON d.customer_code = c.customer_code
			WHERE d.fiscal_year = 2021
			  AND c.market = 'India'
			GROUP BY
			    c.customer_code,
			    c.customer
			ORDER BY average_discount_percentage DESC
			LIMIT 5;

* 7.Get the complete report of the Gross sales amount for the customer “Atliq
		Exclusive” for each month. This analysis helps to get an idea of low and
		high-performing months and take strategic decisions.
		The final report contains these columns:
		Month / Year / Gross sales Amount

			Select 
				monthname(s.date) as Month,
			    year(s.date) as Year,
			    Round(sum((sold_quantity*gross_price)),2) as Gross_sales_amount
			from fact_sales_monthly s
			join fact_gross_price g 
				on s.product_code = g.product_code and
					s.fiscal_year = g.fiscal_year
			join dim_customer c
				on s.customer_code = c.customer_code
			where customer = "Atliq Exclusive" 
			group by 
				Year(s.date),
			    Month(s.date),
			    monthname(s.date)
			order by 
				Year(s.date),
				Month(s.date);

 * 8.In which quarter of 2020, got the maximum total_sold_quantity? The final
		output contains these fields sorted by the total_sold_quantity,
		Quarter / total_sold_quantity

			SELECT
			    CONCAT('Q', quarter_num) AS Quarter,
			    total_sold_quantity
			FROM
			(
			    SELECT
			        QUARTER(date) AS quarter_num,
			        SUM(sold_quantity) AS total_sold_quantity
			    FROM fact_sales_monthly
			    WHERE fiscal_year = 2020
			    GROUP BY QUARTER(date)
			) AS t
			ORDER BY total_sold_quantity DESC;
* 9.Which channel helped to bring more gross sales in the fiscal year 2021
  	and the percentage of contribution? The final output contains these fields,
	channel / gross_sales_mln / percentage

		SELECT
		    c.channel,
		    ROUND(SUM(s.sold_quantity * g.gross_price) , 2) AS gross_sales_mln,
		    ROUND(
		        SUM(s.sold_quantity * g.gross_price) * 100 /
		        (
		            SELECT SUM(s2.sold_quantity * g2.gross_price)
		            FROM fact_sales_monthly s2
		            JOIN fact_gross_price g2
		                ON s2.product_code = g2.product_code
		                AND s2.fiscal_year = g2.fiscal_year
		            WHERE s2.fiscal_year = 2021
		        ),
		        2
		    ) AS percentage
		FROM fact_sales_monthly s
		JOIN fact_gross_price g
		    ON s.product_code = g.product_code
		    AND s.fiscal_year = g.fiscal_year
		JOIN dim_customer c
		    ON s.customer_code = c.customer_code
		WHERE s.fiscal_year = 2021
		GROUP BY c.channel
		ORDER BY percentage DESC;

  * 10.Get the Top 3 products in each division that have a high
 		total_sold_quantity in the fiscal_year 2021? The final output contains these
		fields, division / product_code / product / total_sold_quantity / rank_order

			with product_sales as 
				(Select 
					p.division,
					p.Product_code,
					p.product,
					sum(s.sold_quantity) as Total_sold_quantity
				from fact_sales_monthly s
				join dim_product p
					on p.product_code = s.product_code
				where s.fiscal_year = 2021
				group by 
					p.division,
					p.product_code,
			        p.product
				),
			rank_product as 
					(select 
						division,
						Product_code,
						product,
						Total_sold_quantity,
						rank() over(partition by division order by Total_sold_quantity desc) as rank_order
					from product_sales )
			select * 
			from rank_product
			where rank_order <= 3
			order by division, rank_order;
	
	# Author
    	*Aasmin Khatoon
    	*Aspiring Data Analyst | Excel | Power BI | SQL
