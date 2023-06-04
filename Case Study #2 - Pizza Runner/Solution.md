# 🍕 Case Study #2: Pizza Runner

	Work in Progress!
	Once completed, I'll delete this message :)

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

![image](https://github.com/orseg/8-Week-SQL-Challenge/assets/83500544/1a0b3117-5cb1-4c40-9126-d7b8edaeb0dd)
```sql
declare @week_starts date = '2021-01-01' -- week starts 2021-01-01
select datediff(WK, @week_starts,registration_date) as 'registration_week', count(runner_id) as 'signed_up_runners'
from runners
group by datediff(WK, @week_starts,registration_date)
```

2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

![image](https://github.com/orseg/8-Week-SQL-Challenge/assets/83500544/c6510397-239d-4b86-92e4-0f3d5b9a1a0d)
```sql
-- I'm pretty sure that my answer is not entirely correct but close.
select ro.runner_id, avg(DATEDIFF(MINUTE,co.order_time,ro.pickup_time)) as 'avg_time'
from customer_orders_1 as co
left join runner_orders_1 as ro
on co.order_id = ro.order_id
where ro.duration is not null
group by ro.runner_id
```

3. Is there any relationship between the number of pizzas and how long the order takes to prepare?

![image](https://github.com/orseg/8-Week-SQL-Challenge/assets/83500544/d0af8363-1041-4495-b18b-80b73651dc69)
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
