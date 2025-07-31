# Zomato_Sql_Project
# Zomato Data Analysis - A Food Delivery Company

# Overview

This project comprehensively showcases my SQL problem-solving acumen through in-depth data analysis for Zomato, a prominent food delivery entity in India. The endeavor spans the entire data lifecycle, encompassing database initialization, data ingestion, robust null value management, tackling diverse business challenges via advanced SQL queries, developing efficient stored procedures, optimizing query performance, and ultimately generating impactful insights and actionable recommendations.

## Project Structure

- **Database Setup:** Creation of the `Zomato_db` database and the required tables.
- **Data Import:** Inserting sample data into the tables.
- **Data Cleaning:** Handling null values and ensuring data integrity.
- **Business Problems:** Solving 20 specific business problems using SQL queries.
- **Stored Procedure:** Creating Stored Procedures
- **Query Optimization:** Optimized the queries using index and best practices

## Database Schema and Table Initialization

-- Creation of the core database for the project.
CREATE DATABASE Zomato_db;

-- 1. Defining the essential table structures.
-- Table to store customer information
create table customers (
    customer_id int primary key,
    customer_name varchar(30),
    registration_date date
);

-- Table to store restaurant details
Create table restaurants (
	restaurant_id int primary key,
    restaurant_name varchar(55),
    city varchar(30),
    opening_hours varchar(55)
);

-- Table to store order details and their relationships
create table orders(
    order_id int primary key,
    customer_id int,
    restaurant_id int,
    order_item varchar(55),
    order_date date,
    order_time time,
    order_status varchar(30),
    total_amount float,
    foreign key (customer_id) references customers (customer_id),
    foreign key (restaurant_id) references restaurants (restaurant_id)
);

-- Table for delivery rider details
create table riders(
    rider_id int primary key,
    rider_name varchar(30),
    signup_date date
);

-- Table for delivery status and information
create table deliveries(
    delivery_id int primary key,
    order_id int,
    delivery_status varchar(35),
    delivery_time time,
    rider_id int,
    foreign key (order_id) references orders(order_id),
    foreign key (rider_id) references riders(rider_id)
);


## Data Cleaning and Handling Null Values
```sql
-- Query to check for null values in each table to ensure data quality.
select * from customers
where customer_name is null or registration_date is null;

select * from restaurants
where restaurant_name is null or city is null or opening_hours is null;

select * from orders
where customer_id is null or restaurant_id is null or order_item is null
or order_date is null or order_time is null or order_status is null or
total_amount is null;

select * from riders
where rider_name is null or signup_date is null;

select * from deliveries
where order_id is null or delivery_status is null or delivery_time is null or rider_id is null;
```

## Feature Engineering
```sql
-- Disable safe updates to allow changes to be made to the entire table.
set sql_safe_updates=0;

-- Add a new column to the orders table to categorize orders by time of day.
Alter table orders
add column time_of_day varchar(20);

-- Populate the new 'time_of_day' column based on the 'order_time'.
Update orders
set time_of_day = (
                    case
                        when order_time between "00:00:00" and "12:00:00" then "Morning"
                        when order_time between "12:01:00" and "16:00:00" then "Afternoon"
                        else "Evening"
                      end );

-- Add a new column to the deliveries table for the same categorization.
Alter table deliveries
add column time_of_day varchar(20);

-- Populate the 'time_of_day' column in the deliveries table based on 'delivery_time'.
Update deliveries
set time_of_day = (
                    case
                        when delivery_time between "00:00:00" and "12:00:00" then "Morning"
                        when delivery_time between "12:01:00" and "16:00:00" then "Afternoon"
                        else "Evening"
                      end );

## Business Problems Solved

### Q1. Top 5 Most Frequently Ordered Dishes
-- Write a query to find the top 5 most frequently ordered dishes by the customer "Arjun Mehta" in the last 2 years.
```sql
select customer_name, order_item, count(order_id) as total_orders
from customers inner join orders
on customers.customer_id = orders.customer_id
where customer_name="Arjun Mehta" and year(order_date) >= year(current_date())-2
group by customer_name, order_item
order by count(order_id) desc
limit 5;
```
### Q2. Popular Time Slots
-- Identify the time slots during which the most orders are placed, based on 2-hour intervals.
```sql
select
      case when hour(order_time) between 0 and 1 then "00:00:00 AM-02:00:00AM"
		  when hour(order_time) between 2 and 3 then "02:00:00 AM-04:00:00 AM"
          when hour(order_time) between 4 and 5 then "04:00:00 AM-06:00:00 AM"
          when hour(order_time) between 6 and 7 then "06:00:00 AM-08:00:00 AM"
		  when hour(order_time) between 8 and 9 then "08:00:00 AM-10:00:00 AM"
          when hour(order_time) between 10 and 11 then "10:00:00 AM-12:00:00 PM"
          when hour(order_time) between 12 and 13 then "12:00:00 PM-02:00:00 PM"
          when hour(order_time) between 14 and 15 then "02:00:00 PM-04:00:00 PM"
          when hour(order_time) between 16 and 17 then "04:00:00 PM-06:00:00 PM"
          when hour(order_time) between 18 and 19 then "06:00:00 PM-08:00:00 PM"
          when hour(order_time) between 20 and 21 then "08:00:00 PM-10:00:00 PM"
          when hour(order_time) between 22 and 23 then "10:00:00 PM-00:00:00 AM"
          end as time_slot,
	count(order_id) as total_order
    from orders
    group by time_slot
    order by total_order desc
    limit 1;

    create index idx_customername on customers(customer_name);
```
### Q3. Order Value Analysis
-- Find the average order value (AOV) per customer who has placed more than 750 orders.
-- Return: customer_name, aov (average order value).
```sql
select customers.customer_id,customer_name, round(avg(total_amount),2) as AOV,count(order_id) as total_orders
from orders inner join customers
on orders.customer_id = customers.customer_id
group by customers.customer_id,customer_name
having total_orders>750;
```
### Q4. High-Value Customers
-- List the customers who have spent more than 100K in total on food orders.
-- Return: customer_name, customer_id.
```sql
select customer_name,customers.customer_id, sum(total_amount) as total_order_amt
from orders inner join customers
on orders.customer_id = customers.customer_id
group by customer_name,customers.customer_id
having total_order_amt>100000
order by total_order_amt desc;
```
### Q5. Orders Without Delivery
-- Write a query to find orders that were placed but not delivered.
-- Return: restaurant_name, city, and the number of not delivered orders.
```sql
select restaurants.restaurant_id,restaurant_name,city, sum(case when delivery_status="Not Delivered" then 1 else 0 end ) as not_delivered_orders
from orders inner join restaurants
on orders.restaurant_id = restaurants.restaurant_id
inner join deliveries
on deliveries.order_id = orders.order_id
where order_status = "Completed" and delivery_status="Not Delivered"
group by restaurants.restaurant_id,restaurant_name,city
order by not_delivered_orders desc;

create index idx_orderstatus on orders(order_status);
create index idx_deliverystatus on deliveries(delivery_status);
```
### Q6. Restaurant Revenue Ranking
-- Rank restaurants by their total revenue from the last 2 years.
-- Return: restaurant_name, total_revenue, and their rank within their city.
```sql
select restaurants.restaurant_id, restaurant_name,city, sum(total_amount) as revenue,
dense_rank() over( partition by city order by sum(total_amount) desc ) as rnk
from orders inner join restaurants
on orders.restaurant_id = restaurants.restaurant_id
where year(order_date)=year(current_date())-2
group by restaurants.restaurant_id,restaurant_name,city;

create index idx_orderdate on orders(order_date);
```
### Q7. Most Popular Dish by City
-- Identify the most popular dish in each city based on the number of orders.
```sql
with cte as (
 select city,order_item, count(order_id) as no_of_orders,
 dense_rank() over(partition by city order by count(order_id) desc) as rnk
 from restaurants inner join orders
 on restaurants.restaurant_id = orders.restaurant_id
 group by city,order_item)
 select cte.* from cte
 where rnk=1;
```
### Q8. Customer Churn
-- Find customers who haven’t placed an order in 2024 but did in 2023.
```sql
select customers.customer_id,customer_name
from customers left join orders
on customers.customer_id = orders.customer_id
where  year(order_date)='2023'
and customers.customer_id not in
           ( select customers.customer_id from customers  left join orders
             on customers.customer_id = orders.customer_id where year(order_date)='2024')
group by customers.customer_id,customer_name;
```
### Q9. Cancellation Rate Comparison
-- Calculate the cancellation rate for each restaurant between the 2023 and 2024.
```sql
with cte as (
select restaurants.restaurant_id,restaurant_name, count(order_id) as cancellation_count
from orders inner join restaurants
on restaurants.restaurant_id = orders.restaurant_id
where order_status="Not Fulfilled" and year(order_date) between "2023" and "2024"
group by restaurants.restaurant_id,restaurant_name),

 total_count as (
select restaurants.restaurant_id,restaurant_name, count(order_id) as total_counts
from orders inner join restaurants
on restaurants.restaurant_id = orders.restaurant_id
group by restaurants.restaurant_id,restaurant_name)

select cte.restaurant_id,cte.restaurant_name,
concat(round(cte.cancellation_count * 100 / total_count.total_counts,2)," ","%") as cancellation_rate
from cte inner join total_count
on cte.restaurant_id = total_count.restaurant_id
group by cte.restaurant_id,cte.restaurant_name
order by cancellation_rate desc;
```
### Q10. Rider Average Delivery Time
-- Determine each rider's average delivery time.
```sql
with cte as (
select riders.rider_id,rider_name,
case when delivery_time > order_time then round(timestampdiff(minute,order_time,delivery_time),2)
     else round(timestampdiff(minute,order_time,delivery_time + interval 1 day),2) end as delivery_min
from riders left join deliveries
on riders.rider_id = deliveries.rider_id
inner join orders
on orders.order_id = deliveries.order_id)

select rider_id,rider_name, round(avg(delivery_min),2) as avg_delivery_time_mints
from cte
group by rider_id,rider_name
order by avg_delivery_time_mints desc;
```
### Q11. Monthly Restaurant Growth Ratio
-- Calculate each restaurant's growth ratio for monthly, based on the total number of delivered orders since its joining.
```sql
with cte as(
select r.restaurant_id,restaurant_name,date_format(order_date, "%m/%Y") as month_no,
count(o.order_id) as total_orders,
lag(count(o.order_id) ,1) over(partition by restaurant_id order by date_format(order_date, "%m/%Y")) as previous_month_order
from restaurants r left join orders o
on r.restaurant_id = o.restaurant_id
left join deliveries d
on d.order_id = o.order_id
where delivery_status = "Delivered"
group by r.restaurant_id,restaurant_name,month_no
order by r.restaurant_id,month_no)

select cte.restaurant_id,cte.restaurant_name,cte.month_no, cte.total_orders,previous_month_order,
round((cte.total_orders-previous_month_order)/previous_month_order*100,2) as monthly_growth_ratio
from cte
group by cte.restaurant_id,cte.restaurant_name,cte.month_no, cte.total_orders,
previous_month_order,monthly_growth_ratio;
```
### Q12. Customer Segmentation
-- Segment customers into 'Gold' or 'Silver' groups based on their total spending compared to the average order value (AOV). If a customer's total spending exceeds the AOV,
-- label them as 'Gold'; otherwise, label them as 'Silver'.
-- Return: The total number of orders and total revenue for each segment.
```sql
with cte as (
select customers.customer_id, round( avg(total_amount),2) as aov
from customers inner join orders
group by customers.customer_id),

cust_seg as (
select cte.customer_id,
case when sum(total_amount) > aov then "Gold"
     else "Silver"
     end as customer_segmentation
from cte inner join orders
on cte.customer_id = orders.customer_id
group by customer_id
order by customer_id)

select customer_segmentation,sum(total_amount) as total_revenue, count(order_id) as total_orders
from cust_seg inner join orders
on cust_seg.customer_id = orders.customer_id
group by customer_segmentation;
```
### Q13. Rider Monthly Earnings
-- Calculate each rider's total monthly earnings, assuming they earn 8% of the order amount.
```sql
select riders.rider_id,rider_name, date_format(order_date,"%m/%Y") as month_no,round((sum(total_amount)*0.08),2) as monthly_earnings
from riders left join deliveries
on riders.rider_id = deliveries.rider_id
left join orders
on orders.order_id = deliveries.order_id
where order_status="Completed" and delivery_status="Delivered"
group by riders.rider_id,rider_name,month_no
order by riders.rider_id,month_no;
```
### Q14. Rider Ratings Analysis
-- Find the number of 5-star, 4-star, and 3-star ratings each rider has. Riders receive
-- ratings based on delivery time:
-- ●	5-star: Delivered in less than 30 minutes
-- ●	4-star: Delivered between 30 and 45 minutes
-- ●	3-star: Delivered after 45 minutes
```sql
with cte as (
select riders.rider_id,rider_name,
 case when delivery_time > order_time then round(timestampdiff(minute,order_time,delivery_time),2)
      else round(timestampdiff(minute,order_time,delivery_time + interval 1 day),2) end as delivery_min
from riders left join deliveries
on riders.rider_id = deliveries.rider_id
left join orders
on orders.order_id = deliveries.order_id
where order_status="completed" and delivery_status="delivered"
)
select rider_id, rider_name,
sum(case when delivery_min < 30 then 1 else 0 end) as five_star,
sum(case when delivery_min between 30 and 45 then 1 else 0 end) as four_star,
sum(case when delivery_min>45 then 1 else 0 end) as three_star
from cte
group by rider_id,rider_name
order by rider_id;
```
### Q15. Order Frequency by Day
-- Analyze order frequency per day of the week and identify the peak day for each restaurant.
```sql
with cte as (
select restaurants.restaurant_id,restaurant_name,count(order_id) as total_orders, dayname(order_date) as day_of_week
from orders inner join restaurants
on orders.restaurant_id = restaurants.restaurant_id
where order_status="completed"
group by restaurants.restaurant_id,day_of_week
order by restaurants.restaurant_id),

highest_order_day as (
select restaurant_id, restaurant_name,day_of_week,total_orders,
dense_rank() over(partition by restaurant_id order by total_orders desc) as rnk
from cte)

select restaurant_id, restaurant_name,day_of_week,total_orders from highest_order_day
where rnk=1;
```
### Q16. Customer Lifetime Value (CLV)
-- Calculate the total revenue generated by each customer over all their orders.
```sql
select customers.customer_id,customer_name, sum(total_amount) as CLV
from customers left join orders
on customers.customer_id = orders.customer_id
group by customers.customer_id,customer_name;
```
### Q17. Monthly Sales Trends
-- Identify sales trends by comparing each month's total sales to the previous month.
```sql
with cte as (
select date_format(order_date,"%Y") as year_no, date_format(order_date,"%m") as month_no,
sum(total_amount) as total_sales,
lag(sum(total_amount),1) over ( order by date_format(order_date,"%Y"), date_format(order_date,"%m")) as previous_month_sales
from orders
group by year_no,month_no
order by year_no,month_no)

select year_no,month_no,total_sales, previous_month_sales,concat(round((total_sales-previous_month_sales)/previous_month_sales*100,2)," ","%") as growth_ratio
from cte;
```
### Q18. Rider Efficiency
-- Evaluate rider efficiency by determining average delivery times and identifying those with the lowest and highest averages.
```sql
with cte as (
select r.rider_id,rider_name,
case when delivery_time > order_time then round(timestampdiff(minute,order_time,delivery_time),2)
     else round(timestampdiff(minute,order_time,delivery_time + interval 1 day),2) end as delivery_time_took
from riders r left join deliveries d
on r.rider_id = d.rider_id
left join orders o
on o.order_id = d.order_id
where order_status="completed" and delivery_status="delivered"
group by r.rider_id,rider_name,delivery_time_took),

avg_deliv_time as (
select rider_id,rider_name,avg(delivery_time_took) as avg_delivery_time_took
from cte
group by rider_id,rider_name
)
select min(avg_delivery_time_took) as min_avg_delivery_time,
max(avg_delivery_time_took) as max_avg_delivery_time
from avg_deliv_time;
```
### Q19. Order Item Popularity
-- Track the popularity of specific order items over time and identify seasonal demand spikes.
```sql
with cte as(
 select order_item,count(order_id) as total_orders,
    case
        when month(order_date) between 3 and 5 then "Spring"
        when month(order_date) between 6 and 8 then "Summer"
        when month(order_date) between 9 and 11 then "Autumn"
        else "Winter"
        end as seasons
from orders
where order_status="completed"
group by order_item,seasons),

trending_item as (
select  order_item,total_orders, seasons,dense_rank() over(partition by order_item order by total_orders desc) as rnk
from cte)

select order_item,total_orders,seasons from trending_item where rnk=1;
```
### Q20. City Revenue Ranking
-- Rank each city based on the total revenue for the last year (2023).
```sql
select city, sum(total_amount) as total_revenue,
dense_rank() over( order by sum(total_amount) desc) as rnk
from restaurants r left join orders o
on r.restaurant_id = o.restaurant_id
group by city
having total_revenue is not null;
```

## Stored Procedures

-- 1. Create a stored procedure to return all restaurants in a given city
-- Return : Restaurant_name,cusines type,opening hours
```sql
delimiter //
create procedure restaurant_details ( in city varchar(30))
begin
      select distinct restaurant_name,city, order_item,opening_hours
      from restaurants r inner join orders o
      on r.restaurant_id = o.restaurant_id
      where city=city;
end //
delimiter ;
call restaurant_details('chennai');
```
-- 2. Create a stored procedure to show the full order history of a customer sorted by most recent
-- return customer_name,order_id,restaurant_name, order_date,total amount
```sql
delimiter //
create procedure customer_details( in customer_id int)
begin
      select customer_name,order_id,restaurant_name,order_date,total_amount
      from customers c left join orders o
      on c.customer_id = o.customer_id
      left join restaurants r
      on r.restaurant_id = o.restaurant_id
      where c.customer_id = customer_id
      order by order_date desc;
end //
delimiter ;
call customer_details(1);



```
## Query Optimization 
Indexed customer_name, order_date, order_status, and delivery_status columns to improve query performance.

Avoided using SELECT * by specifying only the necessary columns in select statements.

Leveraged Common Table Expressions (CTEs) and JOINs to handle complex queries more efficiently than using nested subqueries.

Utilized JOINs thoughtfully to minimize redundant data and improve query execution speed.


## Key Insights 

Peak Ordering Time: The majority of orders are placed between 2 PM and 4 PM, with minimal activity observed from 12 AM to 2 AM.

Top Customers: Customers with IDs 6, 7, and 5 are highly valuable, each placing over 750 orders with an average order value of ₹300–₹350.

High Non-Deliveries: Restaurants with IDs 5, 7, and 3 have the highest rates of non-delivered orders.

Seasonal Trends: Spring sees the highest dish sales, while winter has the lowest, suggesting a seasonal fluctuation in food demand.

Underperforming Riders: Riders with IDs 1, 2, 3, and 4 consistently show longer delivery times, which corresponds to them receiving 3-star ratings.

Top-Selling Items by City: Popular dishes like Chicken Biryani, Mutton Rogan Josh, and Paneer Butter Masala are consistently best-sellers across major cities.


## Actionable Recommendations

Introduce Late-Night Offers: Create special discounts or combo deals to encourage more orders during the low-activity period of 12 AM to 2 AM.

Reward Loyal Customers: Implement a VIP program or exclusive deals for high-frequency customers to increase retention and loyalty.

Improve Restaurant Reliability: Conduct audits and provide support to restaurants with high non-delivery rates to help them improve operations and reduce order cancellations.

Optimize Delivery Staff: Offer training or reassign low-rated riders to improve their performance and enhance overall customer satisfaction.

Plan Seasonal Campaigns: Develop season-specific menus and targeted offers for spring to capitalize on peak demand, and create special winter discounts to balance the seasonal drop in sales.

Promote Best-Selling Dishes: Use regional best-selling items in targeted advertising and bundled offers to drive repeat sales.

### *conclusion*

## Conclusion
This project showcases my ability to use advanced SQL queries to solve real-world business problems within a food delivery context. The structured approach demonstrates strong skills in data manipulation and the ability to extract meaningful, actionable insights from data.
