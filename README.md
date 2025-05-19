Coffee Sales SQL Project

Objective

The goal of this project is to analyze the sales data of Coffee, a company that has been selling its products online since January 2023, and to recommend the top three major cities in India for opening new coffee shop locations based on consumer demand and sales performance.

 Questions

1) Coffee Consumers Count
How many people in each city are estimated to consume coffee, given that 25% of the population does?

SELECT city_name,
round((population *0.25)/1000000,2) as coffee_consumer_in_million,
city_rank
from city
order by 2 desc


2) Total Revenue from Coffee Sales
What is the total revenue generated from coffee sales across all cities in the last quarter of 2023?

select 
ci.city_name,
sum(total) as total_revenue
from sales s
join customers c
on s.customer_id = c.customer_id
join city ci
on ci.city_id = c.city_id
where 
extract(year from sale_date) = 2023
and 
extract(quarter from sale_date) = 4
group by 1
order by 2 desc


3)Sales Count for Each Product
How many units of each coffee product have been sold?

select product_name,
count(sale_id) as total_orders
from products p
join sales s
on p.product_id = s.product_id
group by 1
order by 2 desc

4) Average Sales Amount per City
What is the average sales amount per customer in each city?

select city_name,
sum(total) as total_revenue,
round(sum(total)::numeric/count(distinct s.customer_id),2) as avg_sale_per_cust
from sales s
join customers c 
on s.customer_id = c.customer_id
join city ci
on ci.city_id = c.city_id
group by 1
order by 2 desc

5) City Population and Coffee Consumers
Provide a list of cities along with their populations and estimated coffee consumers.

with table1 as
(
SELECT city_name,
round((population *0.25)/100000,2) as coffee_consumer_in_million
from city
),
table2 as
(
select city_name,
count(distinct c.customer_id) as unique_cus
from sales s
join customers c 
on s.customer_id = c.customer_id
join city ci
on ci.city_id = c.city_id
group by 1
)
select t1.city_name,
coffee_consumer_in_million,
unique_cus
from table1 t1
join table2 t2
on t1.city_name = t2.city_name

6) Top Selling Products by City
What are the top 3 selling products in each city based on sales volume?

select city_name,product_name,total_sales
(
select city_name,product_name,
sum(price) as total_sales,
rank() over(partition by city_name order by sum(price)) rn
from products p
join sales s
on p.product_id = s.product_id
join customers c 
on s.customer_id = c.customer_id
join city ci
on ci.city_id = c.city_id
group by 1,2)
where rn <= 3

7) Customer Segmentation by City
How many unique customers are there in each city who have purchased coffee products?

Select city_name,
count(distinct c.customer_id) as unique_customer
from city ci
join customers c
on c.city_id = ci.city_id
join sales s
on c.customer_id = s.customer_id
join products p
on p.product_id = s.product_id
where product_name like '%Coffee%'
group by 1


8) Average Sale vs Rent
Find each city and their average sale per customer and avg rent per customer

select city_name,
round(sum(total)::numeric/count(distinct s.customer_id),2) as avg_sale_per_cust,
round(sum(estimated_rent)::numeric/count(distinct s.customer_id),2) as avg_rent_per_cust
from sales s
join customers c 
on s.customer_id = c.customer_id
join city ci
on ci.city_id = c.city_id
group by 1
order by 2 desc


9) Monthly Sales Growth
Sales growth rate: Calculate the percentage growth (or decline) in sales over different time periods (monthly).

with table1 as
(
select city_name,
extract(month from sale_date) as month,
extract(year from sale_date) as year,
sum(total) as total_sale
from sales s
join customers c
on c.customer_id = s.customer_id
join city ci 
on ci.city_id = c.city_id
group by 1,2,3
order by 1,3,2
),
table2 as
(
select 
city_name,
month,
year,
total_sale as cur_sale,
lag(total_sale,1) over(partition by city_name order by year,month) as last_month_sale
from table1
)
select city_name,
month,
year,
cur_sale,
last_month_sale,
round((cur_sale - last_month_sale)::numeric/last_month_sale::numeric*100,2) as growth_rent
from table2
where last_month_sale is not null


10) Market Potential Analysis
Identify top 3 city based on highest sales, return city name, total sale, total rent, total customers, estimated coffee consumer

with table1 as
(
SELECT city_name,sum(total) as total_sale,
sum(estimated_rent) as total_rent,
count(distinct c.customer_id) as total_customer,
round((population *0.25)/1000000,2) as coffee_consumer_in_million,
rank() over(partition by city_name order by sum(total)desc) as rn
from city ci 
join customers c
on ci.city_id = c.city_id
join sales s 
on c.customer_id = s.customer_id
join products p 
on p.product_id = s.product_id
group by 1,5
)
select 
city_name,total_sale,
total_rent,total_customer,
coffee_consumer_in_million
from table1
where rn = 1


Recommendations
After analyzing the data, the recommended top three cities for new store openings are:

City 1: Pune

Average rent per customer is very low.
Highest total revenue.
Average sales per customer is also high.

City 2: Delhi

Highest estimated coffee consumers at 7.7 million.
Highest total number of customers, which is 68.
Average rent per customer is 330 (still under 500).

City 3: Jaipur

Highest number of customers, which is 69.
Average rent per customer is very low at 156.
Average sales per customer is better at 11.6k.

