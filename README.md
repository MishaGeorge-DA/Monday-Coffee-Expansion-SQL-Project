# Monday-Coffee-Expansion-SQL-Project
The aim of this project is to analyze the sales data of Monday Coffee, an online retailer since January 2023, and identify the top three cities in India for new coffee shop locations based on consumer demand and sales performance.

**Questions for Analysis**

1) How many people in each city are estimated to consume coffee, given that 25% of the population does?
--select city_name, 
concat(round (((0.25*population)/1000000),2),' ', 'M') as Coffee_Consumers 
from city
order by Coffee_Consumers desc;
 
2) What is the total revenue generated from coffee sales across all cities in the last quarter of 2023?
--select 
ci.city_Name, sum(s.total) as Total_Revenue
from sales s
join customers c
on s.customer_id = c.customer_id
join city ci 
on c.city_id = ci.city_id
where sale_date between '2023-10-01' and '2023-12-31'
group by ci.city_name
order by Total_Revenue desc;

3) How many units of each coffee product have been sold?
--select p.product_name, count(p.product_id) as Sold_Count
from sales s
join products p 
on s.product_id = p.product_id
group by p.product_name
order by Sold_Count desc;

4) What is the average sales amount per customer in each city?
--select ci.city_name, count(distinct(c.customer_name)) as Customer_count, 
sum(s.total) as Total_Sales,
round(sum(total)/count(distinct(c.customer_name)),2) as Average_Sales
from
sales s 
join customers c 
on s.customer_id = c.customer_id
join city ci 
on c.city_id = ci.city_id
group by ci.city_name
order by Average_Sales desc;

5) Provide a list of cities along with their populations and estimated coffee consumers.
--select ci.city_name, concat ( (ci.population/1000000),'  ','M') As City_Population_Millions, 
round((ci.population*0.00000025),2) as Coffee_Consumers 
from
sales s 
join customers c 
on s.customer_id = c.customer_id
join city ci 
on c.city_id = ci.city_id
group by ci.city_name,ci.population
order by Coffee_Consumers desc;

6) What are the top 3 selling products in each city based on sales volume?
--With Ranking As
(
select p.product_name,ci.city_name,
count(p.product_id) as Volume,
DENSE_RANK() over (partition by ci.city_name order by count(p.product_id)Desc) as Rank 
from
sales s
join products p 
on s.product_id = p.product_id 
join customers c 
on s.customer_id = c.customer_id
join city ci 
on c.city_id = ci.city_id
group by p.product_name,ci.city_name)
select * from Ranking 
where rank in (1,2,3);
 
7) How many unique customers are there in each city who have purchased coffee products? 
--select  ci.city_name, count(distinct(c.customer_name)) as Unique_Customers
from
sales s
join products p 
on s.product_id = p.product_id 
join customers c 
on s.customer_id = c.customer_id
join city ci 
on c.city_id = ci.city_id
where p.product_id <=14
group by ci.city_name
order by Unique_Customers desc;

8) Find each city and their average sale per customer and avg rent per customer.
--select ci.city_name, round(sum(s.total)/count(distinct(c.customer_id)),2) as Avg_Sales,
ci.estimated_rent/count(distinct(c.customer_id)) As Avg_Rent
from
sales s
join customers c 
on s.customer_id = c.customer_id
join city ci 
on c.city_id = ci.city_id
group by ci.city_name,ci.estimated_rent
order by Avg_Sales desc;

9) Sales growth rate: Calculate the percentage growth (or decline) in sales over different time periods (monthly) by each city.
--with Ratio as
	(select ci.city_name as City_Name,(month(s.sale_date)) as Month,(Year(s.sale_date)) as Year, 
	sum(s.total) as Sale,
	lag(sum(s.total),1) over(PARTITION BY ci.city_name ORDER BY YEAR(s.sale_date), MONTH(s.sale_date)) as LG
	from
	sales s
	join customers c 
	on s.customer_id = c.customer_id
	join city ci 
	on c.city_id = ci.city_id
	group by ci.city_name,month(s.sale_date),year(s.sale_date)
)
select City_Name,MONTH, Year, 
((Sale-LG)/LG)*100 as Growth_Ratio from ratio
where ((Sale-LG)/LG)*100 is not NULL
order by City_Name,Year, Month ;

10) Identify top 3 city based on highest sales, return city name, total sale, total rent, total customers, estimated coffee consumer.
-- select ci.city_name, sum(s.total) as Total_Sales ,
ci.estimated_rent,
count(c.customer_id) as Total_Customers,
concat(round (((0.25*population)/1000000),2),' ', 'M') as Coffee_Consumers 
from
sales s
join products p 
on s.product_id = p.product_id 
join customers c 
on s.customer_id = c.customer_id
join city ci 
on c.city_id = ci.city_id
group by  ci.city_name,ci.estimated_rent, ci.population
order by Total_Sales desc;

**Recommendations**
Based on the data analysis, the top three recommended cities for new store openings are:

1. Pune

Lowest average rent per customer
Highest total revenue
High average sales per customer
2. Delhi

Largest estimated coffee consumer base at 7.7 million
Highest total customer count at 68
Average rent per customer is 330 (well below 500)
3. Jaipur

Highest number of customers at 69
Very low average rent per customer at 156
Strong average sales per customer at 11.6k
