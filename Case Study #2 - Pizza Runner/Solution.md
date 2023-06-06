# 🍕 Case Study #2: Pizza Runner

## Cleaning Data

### runner_orders table
![image](https://github.com/orseg/8-Week-SQL-Challenge/assets/83500544/1af77cc9-7262-451f-b0f7-d36a9ae4d523)

### customer_orders table
![image](https://github.com/orseg/8-Week-SQL-Challenge/assets/83500544/f6900be2-105f-45b0-bad4-9ffdeedadf04)

See this? We need to do something about it

```sql
/*
First step before we can actually go on with the questions is data cleaning (learned the hard way).
-- In runner_orders table, we can see inconsistancy of values (actual null, null as value, empty cells, misspellings..)
-- I'll create new table of runner_orders to not lose the original data and do the cleaning on the new table.
*/
drop table if exists runner_orders_1 
select * into runner_orders_1 
from runner_orders

update runner_orders_1 set cancellation = NULL where cancellation = 'null' or cancellation = ''
update runner_orders_1 set pickup_time = NULL where pickup_time = 'null'
update runner_orders_1 set distance = 
case
	when distance like '%km' then trim('km' from distance)
	when distance = 'null' then null
	else distance
end
update runner_orders_1 set duration = 
case
	when duration like '%minutes' then trim('minutes' from duration)
	when duration like '%mins' then trim('mins' from duration)
	when duration like '%mins' then trim('mins' from duration)
	when duration like '%minute' then trim('minute' from duration)
	when duration = 'null' then null
	else duration
end

-- Same for customer_orders table
drop table if exists customer_orders_1 
select * into customer_orders_1 
from customer_orders

update customer_orders_1 set exclusions = NULL where exclusions = 'null' or exclusions = ''
update customer_orders_1 set extras = NULL where extras = 'null' or extras = ''
```

Now we can get on with the questions!

## A. Pizza Metrics

1. How many pizzas were ordered?

![image](https://github.com/orseg/8-Week-SQL-Challenge/assets/83500544/50c33e82-8534-4104-ac57-26b66e7c7353)
```sql
select count(order_id) as 'orders_count' 
from customer_orders_1
```



2. How many unique customer orders were made?

![image](https://github.com/orseg/8-Week-SQL-Challenge/assets/83500544/b89b56d4-05a8-4769-af6c-df5ee3d1e4d4)
```sql
select count(distinct order_id) as 'unique_orders_count' 
from customer_orders_1 
```

3. How many successful orders were delivered by each runner?

![image](https://github.com/orseg/8-Week-SQL-Challenge/assets/83500544/68f9f810-caf2-4da3-963f-2bfb4592dfb2)
```sql
select runner_id, count(order_id) as 'successful_orders' 
from runner_orders_1 
where duration is not null 
group by runner_id 
```


4. How many of each type of pizza was delivered?

![image](https://github.com/orseg/8-Week-SQL-Challenge/assets/83500544/30d32c07-3534-422e-90c1-51730b2028f1)
```sql
/*
 Had to do a workaround on pizza_name column because the type was text which cannot be used in group by 
 so i casted it to nvarchar
 */
select cast(pn.pizza_name as nvarchar(max)) as 'pizza_name', count(co.pizza_id) 'total_deliveries'
from customer_orders_1 as co
left join runner_orders_1 as ro
on co.order_id = ro.order_id
left join pizza_names as pn
on co.pizza_id = pn.pizza_id
where ro.distance is not null
group by co.pizza_id, cast(pn.pizza_name as nvarchar(max))
```


5. How many Vegetarian and Meatlovers were ordered by each customer?

![image](https://github.com/orseg/8-Week-SQL-Challenge/assets/83500544/1d5e775b-274e-4831-8a3b-7e40252edff2)
```sql
select co.customer_id, cast(pn.pizza_name as nvarchar(max)) as 'pizza_name', count(co.pizza_id) 'total_deliveries'
from customer_orders_1 as co
left join runner_orders_1 as ro
on co.order_id = ro.order_id
left join pizza_names as pn
on co.pizza_id = pn.pizza_id
group by co.customer_id, co.pizza_id, cast(pn.pizza_name as nvarchar(max))
```


6. What was the maximum number of pizzas delivered in a single order?

![image](https://github.com/orseg/8-Week-SQL-Challenge/assets/83500544/d936eb77-e97c-40ff-94a6-cc0c717d49e7)
```sql
select top(1) ro.order_id, count(ro.order_id) as 'max_delivered'
from customer_orders_1 as co
left join runner_orders_1 as ro
on co.order_id = ro.order_id
where ro.duration is not null
group by co.customer_id, ro.order_id
order by count(ro.order_id) desc
```

7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

![image](https://github.com/orseg/8-Week-SQL-Challenge/assets/83500544/ba3fa7d4-8e14-4645-a9a3-dbc2ef69b8c1)
```sql
select co.customer_id, 
sum (case
	when co.exclusions is not null or co.extras is not null then 1
	else 0
end) as 'at_least_1_change', 
sum (case
	when co.exclusions is null and co.extras is null then 1
	else 0
end) as 'no_change'
from customer_orders_1 as co
left join runner_orders_1 as ro
on co.order_id = ro.order_id
where ro.duration is not null
group by co.customer_id
```

8. How many pizzas were delivered that had both exclusions and extras?

![image](https://github.com/orseg/8-Week-SQL-Challenge/assets/83500544/2bc0a0a6-6e31-424d-b03b-d65c9f120b4c)
```sql
select co.customer_id, 
sum (case
	when co.exclusions is not null and co.extras is not null then 1
	else 0
end) as 'both' 
from customer_orders_1 as co
left join runner_orders_1 as ro
on co.order_id = ro.order_id
where ro.duration is not null
group by co.customer_id
order by both desc
```

9. What was the total volume of pizzas ordered for each hour of the day?

![image](https://github.com/orseg/8-Week-SQL-Challenge/assets/83500544/7438aec7-3f2d-456c-b628-1f94592bbc8f)
```sql
select datepart(hour, order_time) as 'hour_of_day', count(pizza_id) as 'total_pizzas_ordered'
from customer_orders_1
group by datepart(hour, order_time)
order by hour_of_day
```

10. What was the volume of orders for each day of the week?

![image](https://github.com/orseg/8-Week-SQL-Challenge/assets/83500544/04c87342-1549-4f10-8742-9095ea106891)
```sql
select datename(WEEKDAY ,order_time) as 'week_day', count(order_id) as 'total_orders'
from customer_orders_1
group by datename(WEEKDAY ,order_time)
order by total_orders desc
```

## B. Runner and Customer Experience

1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

![image](https://github.com/orseg/8-Week-SQL-Challenge/assets/83500544/caf2a2fb-151e-4b00-bcfc-56d803ea9cfc)

```sql
declare @week_starts date = '2021-01-01' -- week starts 2021-01-01
select datediff(WK, @week_starts,registration_date) as 'registration_week', count(runner_id) as 'signed_up_runners'
from runners
group by datediff(WK, @week_starts,registration_date)
```

2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

![image](https://github.com/orseg/8-Week-SQL-Challenge/assets/83500544/64d78c76-2531-4ae0-bd67-10edba30f716)

```sql
-- I'm pretty sure that my answer is not entirely correct but close.
select ro.runner_id, avg(DATEDIFF(MINUTE,co.order_time,ro.pickup_time)) as 'avg_time'
from customer_orders_1 as co
left join runner_orders_1 as ro
on co.order_id = ro.order_id
where ro.distance is not null
group by ro.runner_id
```

3. Is there any relationship between the number of pizzas and how long the order takes to prepare?

![image](https://github.com/orseg/8-Week-SQL-Challenge/assets/83500544/88bd6886-a03c-430c-a2da-82117ea2c43b)

```sql
with cte as 
	(
	select co.customer_id, count(co.customer_id) as 'pizza_count', 
	row_number() over(partition by co.customer_id order by co.customer_id) as row_number, 
	DATEDIFF(MINUTE,co.order_time,ro.pickup_time) as 'avg_time_to_make'
	from customer_orders_1 as co
	left join runner_orders_1 as ro
	on co.order_id = ro.order_id
	where ro.distance is not null
	group by co.customer_id,co.order_time,ro.pickup_time
)
select pizza_count, avg_time_to_make
from cte
where row_number=1
group by pizza_count, avg_time_to_make
```

4. What was the average distance travelled for each customer?

![image](https://github.com/orseg/8-Week-SQL-Challenge/assets/83500544/3ec40cc0-c687-48ed-ae8d-5527560b0161)
```sql
with cte_avg as (select co.customer_id, round(avg(cast(ro.distance as float)),1) as 'avg_distance'
from customer_orders_1 as co
left join runner_orders_1 as ro
on co.order_id = ro.order_id
where ro.distance is not null
group by co.customer_id)
select * from cte_avg;
```

5. What was the difference between the longest and shortest delivery times for all orders?

![image](https://github.com/orseg/8-Week-SQL-Challenge/assets/83500544/8edee17b-a91c-44f4-88d2-ca3783eda8c0)
```sql
with time_diff as (select co.order_id, DATEDIFF(MINUTE,co.order_time,ro.pickup_time) as 'timediff'
from customer_orders_1 as co
inner join runner_orders_1 as ro
on co.order_id = ro.order_id
where ro.distance is not null
group by co.order_id,co.order_time,ro.pickup_time)
select max(timediff) - min(timediff) as 'time_diff'
from time_diff;
```

6. What was the average speed for each runner for each delivery and do you notice any trend for these values?

![image](https://github.com/orseg/8-Week-SQL-Challenge/assets/83500544/499c98fb-4628-4118-9185-addaade694a2)

```sql
select runner_id, order_id, round((cast(distance as float)/(cast(duration as float)/60)),1) as 'avg_speed'
from runner_orders_1
where distance is not null
group by order_id, runner_id, distance, duration
order by runner_id;
```

7. What is the successful delivery percentage for each runner?

![image](https://github.com/orseg/8-Week-SQL-Challenge/assets/83500544/ea1b7a2a-f2a0-4905-b9be-27105e3a9efc)

```sql
with d_perc as (select runner_id, (sum(
	case when distance is not null then 1
	else 0
	end)) as successful, count(runner_id) as total
from runner_orders_1
group by runner_id)
select runner_id, successful, total, (cast(successful as float)/total)*100 as 'delivery_perc'
from d_perc
```

## C. Ingredient Optimisation

1. What are the standard ingredients for each pizza?

![image](https://github.com/orseg/8-Week-SQL-Challenge/assets/83500544/cd781b89-48d7-4156-bc3d-bb860303afca)

```sql
-- This question made me learn how apply works.
with split_cte as (select *
from pizza_recipes
cross apply string_split(cast(pizza_recipes.toppings as nvarchar(max)), ','))
select sc.pizza_id, pn.pizza_name, LTRIM(value) as topping_id, pt.topping_name
from split_cte as sc
left join pizza_toppings as pt
on sc.value = pt.topping_id
left join pizza_names as pn
on sc.pizza_id = pn.pizza_id;
```

2. What was the most commonly added extra?

![image](https://github.com/orseg/8-Week-SQL-Challenge/assets/83500544/1576db6b-4450-4505-a777-ac27b76cdb23)

```sql
with extras_cte as(select b.value from customer_orders_1 as a
cross apply string_split(cast(a.extras as nvarchar(max)), ',') b)
select LTRIM(value) as extra, cast(topping_name as nvarchar(max)) as topping_name, count(LTRIM(value)) as extra_count
from extras_cte
left join pizza_toppings as toppings
on extras_cte.value = topping_id
group by cast(topping_name as nvarchar(max)), LTRIM(value)
order by extra_count desc;
```

3. What was the most common exclusion?

![image](https://github.com/orseg/8-Week-SQL-Challenge/assets/83500544/75e3be94-3050-44d5-93b7-830682b31166)

```sql
with exc_cte as(select b.value from customer_orders_1 as a
cross apply string_split(cast(a.exclusions as nvarchar(max)), ',') b)
select LTRIM(value) as exclusion, cast(topping_name as nvarchar(max)) as topping_name, count(LTRIM(value)) as exc_count
from exc_cte
left join pizza_toppings as toppings
on exc_cte.value = topping_id
group by cast(topping_name as nvarchar(max)), LTRIM(value)
order by exc_count desc;
```

4. Generate an order item for each record in the customers_orders table in the format of one of the following:

* Meat Lovers
	
* Meat Lovers - Exclude Beef
	
* Meat Lovers - Extra Bacon
	
* Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers
    
![image](https://github.com/orseg/8-Week-SQL-Challenge/assets/83500544/588eb289-74ee-44dd-b822-e84afe47e10d)


```sql
-- I tried. Not quiet there. that's quiet an advanced question for my level.
with order_itemz as(select a.order_id, a.customer_id, a.pizza_id,x.value as exc, y.value as ext, order_item =
case
	when exclusions is null and extras is null then (select pizza_name from pizza_names where a.pizza_id = pizza_names.pizza_id)
	when exclusions is not null and extras is null then CONCAT((select pizza_name from pizza_names where a.pizza_id = pizza_names.pizza_id),' ', 
	'- Exclude ', (select cast(topping_name as nvarchar(max)) from pizza_toppings where topping_id in (x.value)))
	when exclusions is null and extras is not null then CONCAT((select pizza_name from pizza_names where a.pizza_id = pizza_names.pizza_id),' ', 
	'- Extra ', (select cast(topping_name as nvarchar(max)) from pizza_toppings where topping_id in (y.value)))
	when exclusions is not null and extras is not null then 
	CONCAT(
		(select pizza_name from pizza_names where a.pizza_id = pizza_names.pizza_id),' ', 
		'- Exclude ', (select cast(topping_name as nvarchar(max)) from pizza_toppings where topping_id in (x.value)), ' ', 
		'- Extra ', (select cast(topping_name as nvarchar(max)) from pizza_toppings where topping_id in (y.value))
	)
end
from customer_orders_1 as a
outer apply string_split(cast(a.exclusions as nvarchar(max)), ',') x
outer apply string_split(cast(a.extras as nvarchar(max)), ',') y
group by a.order_id, a.customer_id, a.pizza_id, a.exclusions, a.extras, x.value, y.value)
select order_id,customer_id,pizza_id, ltrim(exc) as exclusions, ltrim(ext) as extras, cast(order_item as nvarchar(max))
from order_itemz
group by order_id, customer_id, pizza_id,exc,ext, cast(order_item as nvarchar(max))
order by order_id
```

## D. Pricing and Ratings

1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?

![image](https://github.com/orseg/8-Week-SQL-Challenge/assets/83500544/77f00256-d7fa-4830-b640-a879d78844b6)

```sql
select sum(
case
when co.pizza_id = 1 then 12
else 10
end) as total_earn
from customer_orders_1 as co
left join runner_orders_1 as ro
on co.order_id = ro.order_id
where ro.distance is not null;
```

2. What if there was an additional $1 charge for any pizza extras?

![image](https://github.com/orseg/8-Week-SQL-Challenge/assets/83500544/39b6485e-5f5a-40a8-aa81-918181e2643e)

```sql
-- Step 1. create temp table with separated only extras selected (extras not null)
with calc_extras as (select order_id, y.value
from customer_orders_1 as co
cross apply string_split(cast(co.extras as nvarchar(max)), ',') y)
select order_id, value
into #extras_temp 
from calc_extras 

-- Step 2. Sum the current total amount we got earlier with how much extras we got from the tmep table for successful deliveries.
declare @current_total_amount int = '138';
select top(1) total_earnings = sum(cast(ltrim(#extras_temp.value) as int)+@current_total_amount)
from #extras_temp
left join runner_orders_1 as ro
on #extras_temp.order_id = ro.order_id
where ro.distance is not null
group by #extras_temp.order_id, #extras_temp.value
order by total_earnings desc
```

3. The Pizza Runner team now wants to add an additional ratings system that allows 
customers to rate their runner, how would you design an additional table for this new 
dataset - generate a schema for this new table and insert your own data 
for ratings for each successful customer order between 1 to 5.

![image](https://github.com/orseg/8-Week-SQL-Challenge/assets/83500544/2b4e26bc-c30f-41b8-a201-e756417d6d47)


```sql
drop table if exists ratings;
create table ratings(
order_id integer,
rating integer
);
insert into ratings
(order_id, rating)
values
(1,4),
(2,5),
(3,1),
(4,2),
(5,5),
(6,NULL), -- order 6 is cancelled so no rating.
(7,5),
(8,1),
(9,NULL), -- same with order 9.
(10,4);
```

4. Using your newly generated table - can you join all of the information together to 
form a table which has the following information for successful deliveries?

* customer_id
	
* order_id
	
* runner_id
	
* rating
	
* order_time
	
* pickup_time
	
* Time between order and pickup
	
* Delivery duration
	
* Average speed
	
* Total number of pizzas
    
![image](https://github.com/orseg/8-Week-SQL-Challenge/assets/83500544/6eb9691e-82cc-45b8-827b-882f082a672e)

```sql
select co.customer_id, co.order_id, ro.runner_id, ratings.rating, co.order_time, ro.pickup_time, DATEDIFF(MINUTE,co.order_time,ro.pickup_time) as 'time_diff',
ro.duration, round((cast(ro.distance as float)/(cast(ro.duration as float)/60)),1) as 'avg_speed', count(co.customer_id) as 'total_pizzas_delivered'
from customer_orders_1 as co
left join runner_orders_1 as ro
on co.order_id = ro.order_id
left join ratings
on ratings.order_id = co.order_id
where ro.distance is not null
group by co.customer_id, co.order_id, ro.runner_id, ratings.rating, co.order_time, ro.pickup_time, ro.duration, ro.distance
```

5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras 
and each runner is paid $0.30 per kilometre traveled - how much money does Pizza 
Runner have left over after these deliveries?

![image](https://github.com/orseg/8-Week-SQL-Challenge/assets/83500544/9c8ac43d-cf2e-494b-84dd-47331422d9a6)

```sql
declare @earnings_for_fixed_price int = 138;
select leftover_amount =  @earnings_for_fixed_price - sum((0.3*cast(distance as float)))
from runner_orders_1
```
