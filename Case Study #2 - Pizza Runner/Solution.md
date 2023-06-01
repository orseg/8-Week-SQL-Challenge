# 🍕 Case Study #2: Pizza Runner

## Cleaning Data
### customer_orders table
![image](https://github.com/orseg/8-Week-SQL-Challenge/assets/83500544/f6900be2-105f-45b0-bad4-9ffdeedadf04)

### runner_orders table
![image](https://github.com/orseg/8-Week-SQL-Challenge/assets/83500544/1af77cc9-7262-451f-b0f7-d36a9ae4d523)

See this? We need to do something about it

```sql
-- First step before we can actually go on with the questions is data cleaning (learned the hard way).
-- In runner_orders table, we can see inconsistancy of values (actual null, null as value, empty cells, misspelling of minutes..)
-- I'll create new table of runner_orders to retain original data and do the cleaning on the new table.

drop table if exists runner_orders_1 
select * into runner_orders_1 from runner_orders

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
select * into customer_orders_1 from customer_orders

update customer_orders_1 set exclusions = NULL where exclusions = 'null' or exclusions = ''
update customer_orders_1 set extras = NULL where extras = 'null' or extras = ''
```

