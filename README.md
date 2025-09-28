#Problem definition

##Business context: 
HAGUMA car dealership is an automotive business which is specialized in selling cars in Rwanda.
As the car market is becoming competitive, the business is trying to providebrand new electric cars as
they are the future cars, and also to comply with the reduction of emission gases produced by gasoline and diesel cars.

##Data challenge: 
As the company is growing and making sales as usual, the customers are not aware of the electric vehicles. 
In the period of 3 months, the business wants to increase the sales of EVs from 20 cars to 50 cars each month.

##Expected outcome:
As in the month of January, February and March, the business is improving the marketing strategies and the sales of the 
electric vehicles are increasing from 17 cars per month to 20 cars per month, and this indicates that with in a timeframe
of 4 months, the goal to achieve 50 sales of EVs will be achieved.

#Database schema 

Customers(customer_id, first_name, last_name, Email, phone_number, region)
Vehicles(vehicle_id, indentification, manufacturer, model, vehicle_year, power_type, MSRP)
Salesperson(salesperson_id, first_name, last_name, phone_number)
Sales(sales_id, sales_date, vehicle_id, customer_id, salesperson_id, sales_price, quantity)

#Queries used in this database with MySQL 

##Creating tables

create table customers(customer_id varchar(7), First_name varchar(15), 
Last_name varchar(15), Email varchar(30), Phone_number varchar(13) not null,
primary key(customer_id));

create table vehicles(vehicle_id varchar(7), Identification varchar(20), 
Manufacturer text, Model varchar(10), Vehicle_year int, Power_type varchar(10),
MSRP int, primary key(vehicle_id));

create table salesperson(salesperson_id int, First_name varchar(15), 
Last_name varchar(15), Phone_number varchar(13) not null, 
primary key(salesperson_id));

create table sales(sales_id varchar(10), sales_date date, vehicle_id varchar(7), 
customer_id varchar(7), salesperson_id int, sales_price int, quantity int, 
primary key(sales_id), foreign key(vehicle_id) references vehicles(vehicle_id), 
foreign key(customer_id) references customers(customer_id));

##Windows functions queries

-- using ranking windows functions
-- calculating the total_revenue for each customer by joining customer and sales by grouping customers
with customerrevenue as (select c.customer_id, concat(c.first_name,' ', c.last_name) as customer_name, 
sum(s.sales_price) as total_revenue from customers c join sales s on c.customer_id=s.customer_id group by 
c.customer_id, c.first_name, c.last_name) 
-- row_number gives a unique number based on the customer's revenue, dense_rank similar to rank
-- which assigns rank based on revenue
-- percent_rank shows the relative rank of a customer as a percentage
select total_revenue, customer_name, row_number() over(order by total_revenue desc) as
row_numbers, rank() over(order by total_revenue desc) as sales_rank, dense_rank() 
over(order by total_revenue desc) as densesales_rank, percent_rank() over(order by total_revenue desc)
as percentile_rankings from customerrevenue order by total_revenue desc limit 10;

-- aggregate functions
-- using an assumed table dailysales to obtain a single daily revenue
with dailysales as(select sales_date, sum(sales_price) as day_revenue from sales group by 
sales_date order by sales_date)
-- the following query calculates a running total, summing up from the first to the current day
-- the average() over() calculates a weekly moving average
select sales_date, day_revenue, sum(day_revenue) over(order by sales_date rows between unbounded 
preceding and current row) as total_running_revenue, avg(day_revenue) 
over(order by sales_date rows between 6 preceding and current row) as weekly_moving_average, 
max(day_revenue) over(order by sales_date range between interval '29' day preceding and current row)
 as previous_monthly_revenue from dailysales;

--Customer segmentation
with customerrevenue as(select c.customer_id, concat(c.first_name,' ',c.last_name) as customer_name, 
sum(s.sales_price) as total_revenue from customers c join sales s on c.customer_id=s.customer_id 
group by c.customer_id, customer_name)
select customer_name, total_revenue, ntile(4) over(order by total_revenue desc) as quartile_revenue, 
cume_dist() over(order by total_revenue) as cumulative_distribution from customerrevenue 
order by total_revenue desc;

#References:
1. https://www.w3schools.com/sql/
2. https://www.youtube.com/watch?v=OT1RErkfLNQ

“All sources were properly cited. Implementations and analysis represent original work. No AIgenerated
content was copied without attribution or adaptation.”   
