### Customer-Segmentation - Analysis--SQL

## Project Overview

This project demonstrates the use of SQL to perform customer segmentation based on purchasing behavior, demographics, and other relevant data points. Customer segmentation is essential for businesses to personalize marketing, improve customer retention, and optimize product offerings.


## Goals
	•	To segment customers into distinct groups for tailored marketing strategies.
	•	Analyze customer behavior and identify key patterns.
	•	Enable businesses to make data-driven decisions.

## Key Features
 
	1. Data Cleaning and Preprocessing:
	  • Handled missing values and duplicates.
	  •	Standardized formats for consistency.
   
  2.	Segmentation Metrics:
	  •	RFM Analysis (Recency, Frequency, Monetary Value).
	  •	Demographic segmentation (age, location, gender).
	  •	Behavioral segmentation (purchase history, product preferences).

	3.	SQL Queries:
	  •	Complex joins to merge customer, transactions, and product tables.
	  •	Aggregation for RFM scoring.
	  •	Case statements to assign customer categories.

	4.	Insights:
	  •	Identified top-spending customers.
	  •	Segmented customers into tiers (e.g., VIP, Regular, Infrequent).
	  •	Highlighted retention strategies based on data.
   

# Applications

	•	Targeted marketing campaigns.
	•	Improved customer experience through tailored recommendations.
	•	Revenue growth by focusing on high-value segments.

RFM analysis stands for recency, frequency and monetary Value. RFM analysis is a way to use the data based on existing customer behaviour to predict how a new  customer is likely to act in the future. An RFM model is built using three factors:

	1. How recently customer has transacted with a brand
	2. How frequently they've engaged with a brand.
	3. How much money they have spend on brands products and services.

```sql
-- cleaning the data 

select * from Transactions

select * into a_t from Transactions

select * from Transactions
where order_status not like 'Approved'

delete from a_t
where order_status = 'Cancelled'

```

# 1. RFM Analysis

```sql


declare @today_date as date = '2018-01-01';

with base as(
	select  
		 customer_id
		, max(transaction_date) as most_recent_purchase
		, datediff(day, max(transaction_date), @today_date) as recency_score
		, count(transaction_id) as frequency_score
		, sum(list_price) as monetary_value
		, sum(Profit) as monerary_profit
	from a_t
	Group by customer_id
),

rfm_score as (
	select 
		customer_id
		, recency_score
		, frequency_score
		, monetary_value
		, NTILE(5)over(order by recency_score desc) as R 
		, NTILE(5)over(order by frequency_score) as F
		, NTILE(5)over(order by monetary_value) as M
	from base
)

select 
	(R +F+M) / 3 as rfm_group
	, count(rfm.customer_id) as Customer_count
	, cast(sum(monerary_profit) as decimal(16,2)) as rfm_profit
from rfm_score as rfm
inner join base on base.customer_id  = rfm.customer_id
group by (R +F+M) / 3
order by (R +F+M) / 3

-- Output

rfm_group	  Customer_count	  rfm_profit
--------    --------------    ----------
1	              658	        ₹ 9,24,710.79
2	              862	        ₹ 19,56,406.08
3	              1049	      ₹ 35,03,591.53
4	              765         ₹ 35,47,285.03
5	              159	        ₹ 8,99,510.66


```

# 2. Data Analysis and Exploration

# 2.1 New Customer vs Old Customer Age Distributions

```sql
select 
	age
	, tenure
from CustomerDemographic
where tenure >=5 -- where tenure < 5

-- Copying the output and visualizing the data in PowerBI gives the below Visualisation 

```
**Old Customer(tenurity above 5 yrs)**

- The lowest age groups are under 20 and 80+ for custoemrs been with us more than 5 years.
- Among the loyal customers the most populated bracket is the age group 40 - 50.
- Between the age group 40-70 customers stay most loyal to the Brand.

![image](https://github.com/user-attachments/assets/8d5bc476-a754-4e21-bbf3-46f60fa9f1ef)


**New Customers(Tenurity below 5 yrs)**

- Among the New Customers the most populated age bracket is 20-35 and 55- 60.
- There is a steep drop in number of customers in 30-39 age groupsd among the New Customers.
- Youngsters are more prone to become a new customer 

![image](https://github.com/user-attachments/assets/8faeac47-3eb4-4a2c-933c-830754c865fb)

**2.2 Age and Monetary Analysis with RFM Comparison**

```sql
declare @today_date as date = '2018-01-01';

with base as(
	select  
		 customer_id
		, max(transaction_date) as most_recent_purchase
		, datediff(day, max(transaction_date), @today_date) as recency_score
		, count(transaction_id) as frequency_score
		, sum(list_price) as monetary_value
		, sum(Profit) as monerary_profit
	from a_t
	Group by customer_id
),

rfm_score as (
	select 
		customer_id
		, recency_score
		, frequency_score
		, monetary_value
		, NTILE(5)over(order by recency_score desc) as R 
		, NTILE(5)over(order by frequency_score) as F
		, NTILE(5)over(order by monetary_value) as M
	from base
)

select 
	rfm_score.M
	, Age
	, monetary_value
	, rfm_score.customer_id
from rfm_score inner join CustomerDemographic on rfm_score.customer_id = CustomerDemographic.customer_id


```

![image](https://github.com/user-attachments/assets/650f2298-ee72-4c71-ba72-5290731c821c)

- The highest spendings is in the age bracket 40 - 50
- Followed by the age groups 30-40 and 50-60
- Younger Custoemrs and older customers tend to spend less

**2.3 Male Female distribution in rfm group**

```sql

declare @today_date as date = '2018-01-01';

with base as(....

rfm_score as (....

with CTE as (
	select 
		(R+F+M)/3 as RFM_Score
		, gender
	from rfm_score join CustomerDemographic on rfm_score.customer_id = CustomerDemographic.customer_id
)

select 
	RFM_Score
	, gender
	, count(gender) as number_gender
from CTE
group by  RFM_Score,gender
order by RFM_Score

-- Output

RFM_Score	    gender	    number_gender
---------     ------      -------------
1	            Male	          304
1            	Female	        340
2	            Female	        443
2	            Male	          405
3	            Male	          506
3	            Female	        522
4	            Male	          369
4	            Female	        377
5	            Female	         77
5	            Male	           72

```

![image](https://github.com/user-attachments/assets/89c1c477-596a-40ba-b6ad-8ab531fd502f)


**2.4 Job Industry Customer Distribution**









