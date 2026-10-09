# consumer-goods-sql-analysis
SQL-based ad hoc analysis of AtliQ Hardware's consumer goods business, covering product portfolio, manufacturing costs, customer discounts, sales trends, and channel performance to generate actionable business insights.

# Project Overview

* This project focuses on solving ad hoc business requests for AtliQ Hardware, a global consumer electronics company. The objective is to analyze business data using SQL and present meaningful insights that can support data-driven decision-making.

The analysis covers product portfolio changes, manufacturing costs, customer discounts, sales performance, and channel contribution.

# Tools & Technologies
* SQL / MySQL: Business data querying and analysis
* PDF / PowerPoint: Presentation of findings and business insights

# Requests:
# 1.Provide the list of markets in which customer "Atliq Exclusive" operates its business in the APAC region
		SELECT 
		market
		FROM gdb023.dim_customer
		where customer = "Atliq Exclusive" and region = "APAC";
      

   Author
    Aasmin Khatoon
    Aspiring Data Analyst | Excel | Power BI | SQL
